<%*
/* === Move & Link (Create real MOC files: base + sub) ===================== */

/* 0) 목적지 폴더 목록 */
const TARGETS = [
  "01_Java/Core",
  "01_Java/Algorithm",
  "01_Java/Spring",
  "02_Database/PostgreSQL",
  "02_Database/Redis",
  "02_Database/Core",
  "03_CS/Network",
  "03_CS/OS",
  "04_DevOps/Docker"
];

const _target = await tp.system.suggester(TARGETS, TARGETS);
if (!_target) { tR += "취소됨"; return; }

/* 1) 이동 */
await app.vault.createFolder(_target).catch(() => {}); // 폴더 보장
const _newPathNoExt = `${_target}/${tp.file.title}`;
const _newPath = _newPathNoExt + ".md";

// 동일 이름 파일 존재 시, 안전 취소
const _existing = app.vault.getAbstractFileByPath(_newPath);
if (_existing) {
  tR += `⚠️ 동일 이름 파일 존재: ${_newPath} — 이동 취소`;
  return;
}

await tp.file.move(_newPathNoExt);

/* 파일 핸들 재획득 (리트라이) */
let _tfile = null;
for (let i = 0; i < 10; i++) {
  await new Promise(r => setTimeout(r, 100));
  _tfile = app.vault.getAbstractFileByPath(_newPath);
  if (_tfile) break;
}
if (!_tfile) { tR += "❌ 파일 핸들 획득 실패"; return; }

/* 2) 라벨 파싱 */
const [_areaKey, _subRaw] = _target.split("/");
const _areaLabel = (_areaKey || "").replace(/^\d+_/, "").replace(/_/g, " ");
const _subLabel  = _subRaw ? _subRaw.replace(/_/g, " ") : null;

/* 3) 파일형 MOC 경로 */
const _MOC_DIR  = "MOCs";
const _basePath = `${_MOC_DIR}/${_areaLabel}.md`;
const _subDir   = `${_MOC_DIR}/${_areaLabel}`;
const _subPath  = _subLabel ? `${_subDir}/${_subLabel}.md` : null;

/* 4) 유틸 */
async function _ensureFolder(path) {
  try { await app.vault.createFolder(path); } catch (_) {}
}
async function _ensureFile(path, content) {
  let f = app.vault.getAbstractFileByPath(path);
  if (!f) {
    await app.vault.create(path, content);
    f = app.vault.getAbstractFileByPath(path);
  }
  return f;
}

/* 5) MOC 파일 보장 생성 */
await _ensureFolder(_MOC_DIR);
await _ensureFile(_basePath, `# ${_areaLabel}\n\n`);

let _subFile = null;
if (_subPath) {
  await _ensureFolder(_subDir);
  _subFile = await _ensureFile(_subPath, `# ${_subLabel}\n\n`);

  // 하위 MOC → 상위 MOC 역링크 보장 (정확 경로 검사)
  let _s = await app.vault.read(_subFile);
  const _parentLink = `상위 주제: [[${_basePath.replace(/\.md$/, "")}]]`;
  const _parentRegex = new RegExp(
    `상위 주제:\\s*\\[\\[${_basePath.replace(/\.md$/, "").replace(/[.*+?^${}()|[\]\\]/g, "\\$&")}\\]\\]`
  );

  if (!_parentRegex.test(_s)) {
    if (/^# .+\n?/.test(_s)) {
      _s = _s.replace(/^# .+\n?/, (m)=> m + _parentLink + "\n\n");
    } else {
      _s = _parentLink + "\n\n" + _s;
    }
    await app.vault.modify(_subFile, _s);
  }
}

/* 6) 본문 ‘관련 주제’ / 완료 로그 처리
   - 기존 ‘관련 주제’ 라인(인용/비인용) 전부 제거
   - 제목(H1) 바로 아래에 인용문(>) 스타일로 1개만 삽입
   - 문서 맨 아래 ‘이동 완료’ 로그(YYYY-MM-DD) 추가, 위에 빈 줄 1개 보장
*/
const _primary   = _subFile ? `[[${_subPath.replace(/\.md$/, "")}]]`
                            : `[[${_basePath.replace(/\.md$/, "")}]]`;
const _linkLineQ = `> 관련 주제: ${_primary}`;

const _prettyPath = _subLabel ? `${_areaLabel} → ${_subLabel}` : `${_areaLabel}`;
const _today      = tp.date.now("YYYY-MM-DD");
const _doneLine   = `✅ 이동 완료: ${_prettyPath} (${_today})`;

let _full = await app.vault.read(_tfile);

/* 프론트매터 분리 */
let _fm = "", _body = _full;
const _m = _full.match(/^---\s*\n[\s\S]*?\n---\s*\n?/);
if (_m && _m.index === 0) { _fm = _m[0]; _body = _full.slice(_fm.length); }

/* 기존의 모든 ‘관련 주제’ 라인 제거(인용/비인용 전부) */
const _reAnyTopicLine = /^[ \t]*>?[ \t]*\*?\*?관련 주제:?\*?\*?[ \t]*.*$/gm;
let _cleaned = _body.replace(_reAnyTopicLine, "");

/* 과도한 빈 줄 정리 */
_cleaned = _cleaned
  .replace(/^\s+$/gm, "")     // 공백만 있는 줄 제거
  .replace(/\n{3,}/g, "\n\n") // 3줄 이상 공백 → 2줄
  .replace(/^\n+/, "");       // 맨 앞 빈 줄 제거

/* 제목(H1) 바로 아래에 인용문(>) 스타일의 ‘관련 주제’ 삽입 */
const _h1Re = /^# .+$/m;
const _h1Match = _h1Re.exec(_cleaned);
let _newBody;

if (_h1Match) {
  const _insertIdx = _h1Match.index + _h1Match[0].length;
  _newBody =
    _cleaned.slice(0, _insertIdx) +
    `\n\n${_linkLineQ}\n\n` +              // H1 바로 아래
    _cleaned.slice(_insertIdx).replace(/^\n+/, "");
} else {
  // H1이 없으면 맨 앞에
  _newBody = `${_linkLineQ}\n\n${_cleaned}`;
}

/* 문서 맨 아래 ‘이동 완료’ 로그: 중복 방지 + 위에 빈 줄 1개 보장 */
if (!_newBody.includes(_doneLine)) {
  _newBody = _newBody.replace(/\s*$/, "");
  if (!/\n\n$/.test(_newBody)) {
    _newBody += _newBody.endsWith("\n") ? "\n" : "\n\n";
  }
  _newBody += `${_doneLine}\n`;
}

/* 변경 시 저장 */
if (_newBody !== _body) {
  await app.vault.modify(_tfile, _fm + _newBody);
}

/* 7) status: done + updated 자동 갱신 */
try {
  await app.fileManager.processFrontMatter(_tfile, (y) => {
    y.status  = "done";
    y.updated = tp.date.now("YYYY-MM-DD HH:mm");
  });
} catch (e) {
  console.log("status/update 업데이트 실패(무시 가능):", e);
}

/* 8) 완료 토스트 (문서에 로그가 있으니 짧게) */
/*tR += "✅ 완료";*/
%>
