```dataviewjs
/***** 설정 *****/
const FOLDER = "02_Database/Core";

/***** done 판정 규칙 (필요시 커스텀) *****/
const DONE_FIELD = "done";                    // 불리언 done 필드
const DONE_VALUES = ["done", "완료", "complete"]; // status 문자열이 이 값이면 완료로 처리

/***** AND 토글 아이콘 *****/
const ICON_ON   = "☑️";
const ICON_OFF  = "⬜";

/***** 중복 방지: 이전 렌더 전부 제거 후 시작 *****/
const ROOT_ID = "dvjs-tag-filter-stable";
document.querySelectorAll(`#${ROOT_ID}`).forEach(n => n.remove());

/***** 루트 / UI / 결과 컨테이너 *****/
const root = dv.el("div", "", { attr: { id: ROOT_ID } });
const ui   = root.createDiv({ cls: "dvjs-ui" });
const left = ui.createDiv({ cls: "dvjs-ui-left" });   // 드롭다운 / AND / 초기화
const res  = root.createDiv({ cls: "dvjs-result" });

/***** 데이터 로드 *****/
const pages = dv.pages(`"${FOLDER}"`)
  .where(p => p.file.path != dv.current().file.path)
  .sort(p => p.file.mtime, 'desc');

/***** 태그 목록 수집 *****/
const allTags = Array.from(new Set(pages.flatMap(p => p.file.tags ?? []))).sort();

/***** 유틸 *****/
// 기존 fmtDate를 아래로 교체
const fmtDate = (v) => {
  try {
    if (!v) return "-";
    if (typeof v?.toFormat === "function") return v.toFormat("yy-MM-dd"); // Dataview DateTime
    const d = dv.date(v);                      // 문자열/Date도 수용
    return d?.toFormat ? d.toFormat("yy-MM-dd") : "-";
  } catch { return "-"; }
};

function tagCell(tags = []) {
  if (!tags?.length) return "-";
  const wrap = document.createElement("div");
  tags.forEach((t, i) => {
    const a = document.createElement("a");
    a.textContent = t;
    a.href = t;                 // Obsidian 태그 링크
    a.classList.add("tag");
    wrap.appendChild(a);
    if (i < tags.length - 1) wrap.appendChild(document.createElement("br")); // 세로 줄바꿈
  });
  return wrap;
}
function linkCell(p) {
  const td = document.createElement("td");
  const a  = document.createElement("a");

  a.textContent = p.file.name;          // 표시 텍스트
  a.href = p.file.path;                 // 시각적 링크
  a.classList.add("internal-link");     // Obsidian 내부 링크 스타일

  a.onclick = (e) => {
    e.preventDefault();
    const newLeaf = e.ctrlKey || e.metaKey || e.shiftKey; // Ctrl/⌘/Shift 클릭 시 새 창
    try {
      app.workspace.openLinkText(p.file.path, dv.current().file.path, newLeaf);
    } catch (_) {
      console.warn("openLinkText failed for:", p.file.path);
    }
  };

  td.appendChild(a);
  return td;
}

/***** 드롭다운(버튼 + 팝오버) *****/
const ddBtn = left.createEl("button", { text: "태그 선택 (0)" });
ddBtn.classList.add("dvjs-ddbtn");

const pop = left.createDiv({ cls: "dvjs-popover" });
pop.style.display = "none";

const cbWrap = pop.createDiv({ cls: "dvjs-cbwrap" });
const checks = allTags.map(tag => {
  const lbl = cbWrap.createEl("label", { cls: "dvjs-item" });
  const cb  = lbl.createEl("input", { type: "checkbox", value: tag });
  lbl.createSpan({ text: " " + tag });
  return cb;
});

/***** 모두포함(AND) 이모지 토글 버튼 *****/
let useAnd = true;
const andBtn = left.createEl("button", { text: `모두 포함 ${ICON_ON}` });
andBtn.classList.add("dvjs-andbtn");
andBtn.addEventListener("click", () => {
  useAnd = !useAnd;
  andBtn.textContent = `모두 포함 ${useAnd ? ICON_ON : ICON_OFF}`;
  render();
});

/***** 초기화 버튼 — 모두포함 오른쪽에 배치 *****/
const clearBtn = left.createEl("button", { text: "초기화" });
clearBtn.addEventListener("click", () => { checks.forEach(c => c.checked = false); render(); });

/***** 팝오버 열고 닫기 *****/
function openPop() { pop.style.display = "block"; ddBtn.classList.add("open"); }
function closePop() { pop.style.display = "none"; ddBtn.classList.remove("open"); }
ddBtn.onclick = (e) => { e.stopPropagation(); (pop.style.display === "none") ? openPop() : closePop(); };
document.addEventListener("click", (e) => { if (!root.contains(e.target)) closePop(); });

/***** 선택 상태 읽기 *****/
function getChosen() {
  return checks.filter(c => c.checked).map(c => c.value);
}

/***** 상태 표시: 속성값 보존, 화면에만 표시 *****/
const statusIcon = (p) => {
  // 1) 불리언 done 필드 우선
  if (p[DONE_FIELD] === true) return "✅";
  if (p[DONE_FIELD] === false) return "⬜";

  // 2) 문자열 status 판정
  const s = String(p.status ?? "").trim().toLowerCase();
  if (DONE_VALUES.includes(s)) return "✅";

  // 3) 아무것도 없으면 빈 박스
  return "⬜";
};

/***** 표 DOM 순수 렌더 (중복 없음, dv.* 미사용) *****/
function render() {
  const chosen = getChosen();

  // 필터
  const filtered = pages.where(p => {
    const tags = p.file.tags ?? [];
    if (!chosen.length) return true;
    return useAnd ? chosen.every(t => tags.includes(t))
                  : chosen.some(t => tags.includes(t));
  });

  ddBtn.textContent = `태그 선택 (${chosen.length})`;

  // 결과 컨테이너 비우고 새 표 작성
  res.innerHTML = "";

  // 표 생성 (순수 DOM)
  const table = document.createElement("table");
  table.classList.add("dataview", "table-view-table");

  // 헤더
  const thead = document.createElement("thead");
  const trh = document.createElement("tr");
  const headers = [`노트 (${filtered.length})`, "태그", "출처", "수정일", "상태"];
  headers.forEach(h => {
    const th = document.createElement("th");
    th.textContent = h;
    trh.appendChild(th);
  });
  thead.appendChild(trh);
  table.appendChild(thead);

  // 바디
  const tbody = document.createElement("tbody");
  for (const p of filtered) {
    const tr = document.createElement("tr");

    // 1) 노트 링크 (내부 링크)
    tr.appendChild(linkCell(p));

    // 2) 태그 (세로 칩)
    const tdTags = document.createElement("td");
    const tagEl = tagCell(p.file.tags);
    if (typeof tagEl === "string") tdTags.textContent = tagEl;
    else tdTags.appendChild(tagEl);
    tr.appendChild(tdTags);

    // 3) 출처
    const tdSrc = document.createElement("td");
    tdSrc.textContent = p.source ?? "-";
    tr.appendChild(tdSrc);

    // 4) 수정일 (updated 우선)
    const tdDate = document.createElement("td");
	tdDate.textContent = fmtDate(p.file.mtime);
    tr.appendChild(tdDate);

    // 5) 상태 — 표시만 (토글 없음, 원본 값 보존)
    const tdStatus = document.createElement("td");
    tdStatus.style.whiteSpace = "nowrap";
    tdStatus.textContent = statusIcon(p);
    tr.appendChild(tdStatus);

    tbody.appendChild(tr);
  }
  table.appendChild(tbody);
  res.appendChild(table);
}

/***** 이벤트 *****/
checks.forEach(cb => cb.addEventListener("change", render));

/***** 최초 렌더 *****/
render();

/***** 스타일: 정렬/간격 보강 *****/
const style = document.createElement("style");
style.textContent = `
/* ── 컨트롤 바 ─────────────────────────────────────────────── */
#${ROOT_ID} .dvjs-ui {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 10px;
}
#${ROOT_ID} .dvjs-ui-left {
  position: relative;
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

/* 드롭다운 버튼 */
#${ROOT_ID} .dvjs-ddbtn { padding: 6px 10px; border-radius: 8px; }
#${ROOT_ID} .dvjs-ddbtn.open { box-shadow: 0 0 0 2px var(--interactive-accent); }

/* 팝오버 */
#${ROOT_ID} .dvjs-popover {
  position: absolute; top: 36px; left: 0; z-index: 50;
  padding: 10px; background: var(--background-secondary);
  border: 1px solid var(--background-modifier-border);
  border-radius: 12px; box-shadow: var(--shadow-s);
  min-width: 240px; max-height: 260px; overflow: auto;
}
#${ROOT_ID} .dvjs-cbwrap { display: flex; flex-direction: column; gap: 6px; margin-bottom: 8px; }

/* 모두포함 이모지 토글 버튼 */
#${ROOT_ID} .dvjs-andbtn {
  padding: 6px 10px;
  border-radius: 8px;
}

/* ── 표 & 태그 간격 ─────────────────────────────────────── */
#${ROOT_ID} .dvjs-result table.table-view-table td { vertical-align: top; }

/* 태그 칩 간격(겹침 방지) */
#${ROOT_ID} .tag {
  display: inline-block;
  margin: 2px 4px !important;   /* 위아래 2px, 좌우 4px */
  line-height: 1.5;
  padding: 2px 6px;
  border-radius: 6px;
}
`;
root.appendChild(style);

```