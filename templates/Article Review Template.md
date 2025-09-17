<%*
const version = "0.0.1"

const doi = await tp.system.prompt("DOI (or full https://doi.org/…)","10.1016/j.apergo.2021.103607");
if (!doi) { return; }
console.log(doi);
/* ----------  1.  GET BIBTEX FROM DOI.ORG  ---------- */
let bibtex = "";
try {
  const cleanDOI = doi.replace(/^https?:\/\/(dx\.)?doi\.org\//i,"");
  const res      = await fetch(`https://doi.org/${cleanDOI}`, {
                       headers: { Accept: "application/x-bibtex" }
                     });
  if (!res.ok) throw new Error("DOI resolution failed");
  bibtex = await res.text();
} catch (e) {
  new Notice("Could not fetch BibTeX for that DOI – check network/DOI");
  console.error(e);
}

/* ----------  2.  PARSE BIBTEX FIELDS  ---------- */
const getBibField = (field) => {
  const m = bibtex.match(new RegExp(`\\s*${field}\\s*=\\s*\\{(.+?)\\}`, "im"));
  return m ? m[1].trim() : "";
};
const title  = getBibField("title");
const year   = getBibField("year");
const author = getBibField("author").replace(/\s+and\s+/gi,", ");
console.log(title)
/* ----------  3.  RENAME THE NOTE  ---------- */
const safeTitle = (title || "Untitled").replace(/[\\/:*?"<>|]/g,"");
const folderName  = `${year ? year + " – " : ""}${safeTitle}`;
const baseFolder = `${tp.file.folder(true)}/${folderName}`;
const assetsFolder = `${baseFolder}/assets`; 

/* ----------  4.  CREATE FOLDERS  ---------- */
await app.vault.createFolder(baseFolder).catch(() => {});
await app.vault.createFolder(assetsFolder).catch(() => {});

/* ----------  5.  MOVE CURRENT NOTE → index.md  ---------- */
const newPath = `${baseFolder}/review`; 
await tp.file.move(newPath);

/* ----------  6.  WRITE THE NOTE  ---------- */
tR += `---
type: article-review
doi: "${doi}"
title: "${title}"
authors: "${author}"
year: ${year}
created_at: ${tp.date.now("DD-MM-YYYY")}
template_version: ${version}
tags:
---

# ${title}
*${author}${year ? " (" + year + ")" : ""}*

---

## BibTeX
\`\`\`bibtex
${bibtex}
\`\`\`

## Key problem

## Tools

## Experiment

## Methods

---
`;
%>
