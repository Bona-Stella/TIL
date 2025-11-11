## 1) 미분류 · 초안(TIL) 목록
```dataview
TABLE WITHOUT ID
  file.link AS "노트",
  choice(length(file.tags) = 0, "-", join(file.tags, ", ")) AS "태그",
  choice(source, source, "-") AS "출처",
  dateformat(file.ctime, "yy-MM-dd") AS "Created"
FROM "05_Inbox"
WHERE (
  !status
  OR lower(status) = "draft"
  OR contains(lower(status), "draft")
)
AND file.path != this.file.path
SORT file.mtime DESC
```

## 2) 메타데이터 누락 체크
```dataview
TABLE WITHOUT ID
  file.link AS "노트",
  choice(
    (!source) and ((length(file.tags) = 0) and (length(default(tags, [])) = 0)),
    "source, tags",
    choice(
      !source,
      "source",
      choice(
        (length(file.tags) = 0) and (length(default(tags, [])) = 0),
        "tags",
        "-"
      )
    )
  ) AS "누락"
FROM "05_Inbox"
WHERE (
  !source
  OR ((length(file.tags) = 0) and (length(default(tags, [])) = 0))
)
AND file.path != t

```

## 3) 최근 수정 TOP 20 (전체)
```dataview
TABLE WITHOUT ID
  file.link AS "노트",
  file.folder AS "Folder",
  dateformat(file.mtime, "yy-MM-dd") AS "Updated"
FROM ""
WHERE file.path != this.file.path
AND !startswith(file.path, "00_Templates/")
AND !startswith(file.path, "MOCs/")
AND file.name != "Database View"
SORT file.mtime DESC
LIMIT 20
```

## 4) 리뷰 큐 (7일 넘은 초안)
```dataview
TABLE WITHOUT ID
  file.link AS "노트",
  default(date(created), file.ctime) AS "Created"
FROM "05_Inbox"
WHERE (
  !status
  OR status = "draft"
  OR contains(status, "draft")
)
AND (date(now) - default(date(created), file.ctime)) > dur(7 days)
AND file.path != this.file.path
SORT default(date(created), file.ctime) ASC
```