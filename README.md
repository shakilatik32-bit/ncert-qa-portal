/* NCERT QA Portal — Single-file React component

Tailwind CSS utility classes used (no import required in canvas preview)

Default export is App component

Features:

Year toggle (1st Year / 2nd Year)

Subjects -> Chapters -> Questions list

Search across questions and answers

Extra questions section

Import/Export JSON for content

Print / Export to PDF (using window.print fallback; jsPDF commented)

Simple admin add question form (local state only)



How to use:

Drop this file into a React project (e.g., Vite + React, or Next.js page) with Tailwind configured.

For a quick preview, paste into an online React playground that supports Tailwind.


This file is intentionally self-contained and uses an in-memory sample dataset. Replace SAMPLE_DATA with your full NCERT-aligned Q&A dataset. */

import React, { useMemo, useState } from 'react'

const SAMPLE_DATA = { year1: { subjects: [ { id: 'eng', name: 'English', chapters: [ { id: 'eng-ch1', title: 'Prose: The Ailing Planet', questions: [ { id: 'q1', q: 'What is the central theme of the chapter?', a: 'The central theme is environmental degradation...' , tags:['short']}, { id: 'q2', q: 'Explain the author's view on development vs environment.', a: 'The author argues development must be sustainable...' , tags:['long','extra']} ] } ] }, { id: 'math', name: 'Mathematics', chapters: [ { id: 'm-ch1', title: 'Sets and Functions', questions: [ { id: 'm-q1', q: 'Define a set with example.', a: 'A set is a collection of distinct objects...' , tags:['definition']}, { id: 'm-q2', q: 'Prove that intersection is commutative.', a: 'Proof: Let A and B be sets... (step-by-step)' , tags:['proof','extra']} ] } ] } ] }, year2: { subjects: [ { id: 'phy', name: 'Physics', chapters: [ { id: 'p-ch1', title: 'Motion', questions: [ { id: 'p-q1', q: 'State Newton's first law.', a: 'An object remains at rest or in uniform motion unless acted upon by a net external force.' , tags:['short']}, ] } ] } ] } }

export default function App() { const [year, setYear] = useState('year1') const [data, setData] = useState(SAMPLE_DATA) const [subjectId, setSubjectId] = useState(null) const [chapterId, setChapterId] = useState(null) const [query, setQuery] = useState('') const [showExtraOnly, setShowExtraOnly] = useState(false)

const years = { year1: '1st Year', year2: '2nd Year' }

const subjects = useMemo(() => data[year].subjects, [data, year])

function selectSubject(id) { setSubjectId(id) setChapterId(null) }

function selectChapter(id) { setChapterId(id) }

function filteredQuestions() { const subj = subjects.find(s => s.id === subjectId) if (!subj) return [] const chap = subj.chapters.find(c => c.id === chapterId) if (!chap) return [] return chap.questions.filter(q => { if (showExtraOnly && !q.tags?.includes('extra')) return false if (!query) return true const text = ${q.q} ${q.a}.toLowerCase() return text.includes(query.toLowerCase()) }) }

function addQuestion({subjectId: sId, chapterId: cId, qText, aText, tags = []}){ setData(prev => { const copy = JSON.parse(JSON.stringify(prev)) const subj = copy[year].subjects.find(s => s.id === sId) if (!subj) return prev const chap = subj.chapters.find(c => c.id === cId) if (!chap) return prev const id = ${cId}-q${Date.now()} chap.questions.push({id, q: qText, a: aText, tags}) return copy }) }

function handleImportJson(ev) { const file = ev.target.files?.[0] if (!file) return const reader = new FileReader() reader.onload = e => { try { const parsed = JSON.parse(e.target.result) setData(parsed) alert('Imported JSON dataset successfully') } catch (err) { alert('Invalid JSON') } } reader.readAsText(file) }

function handleExportJson() { const blob = new Blob([JSON.stringify(data, null, 2)], {type: 'application/json'}) const url = URL.createObjectURL(blob) const a = document.createElement('a') a.href = url a.download = 'ncert_qa_dataset.json' a.click() URL.revokeObjectURL(url) }

function handlePrint() { window.print() }

return ( <div className="min-h-screen bg-slate-50 p-6"> <div className="max-w-6xl mx-auto bg-white shadow rounded-lg overflow-hidden"> <header className="p-6 flex items-center justify-between"> <div> <h1 className="text-2xl font-bold">NCERT Q&A Portal</h1> <p className="text-sm text-slate-500">1st & 2nd Year — NCERT-aligned questions + extras</p> </div>

<div className="flex gap-2 items-center">
        <div className="inline-flex rounded-md bg-slate-100 p-1">
          {Object.entries(years).map(([k, label]) => (
            <button key={k} onClick={() => setYear(k)} className={`px-3 py-1 rounded ${year===k? 'bg-white shadow' : 'opacity-80'}`}>
              {label}
            </button>
          ))}
        </div>
        <button onClick={handleExportJson} className="btn px-3 py-2 rounded bg-indigo-600 text-white">Export</button>
        <label className="btn px-3 py-2 rounded bg-slate-100 cursor-pointer">
          Import
          <input type="file" accept="application/json" onChange={handleImportJson} className="hidden" />
        </label>
        <button onClick={handlePrint} className="px-3 py-2 rounded bg-emerald-600 text-white">Print</button>
      </div>
    </header>

    <main className="grid grid-cols-1 md:grid-cols-4 gap-6 p-6">
      <aside className="md:col-span-1">
        <div className="mb-4">
          <h3 className="font-semibold">Subjects</h3>
          <ul className="mt-2 space-y-2">
            {subjects.map(s => (
              <li key={s.id} className={`p-2 rounded cursor-pointer ${subjectId===s.id? 'bg-indigo-50' : 'hover:bg-slate-50'}`} onClick={() => selectSubject(s.id)}>
                {s.name}
                <div className="text-xs text-slate-400">{s.chapters.length} chapters</div>
              </li>
            ))}
          </ul>
        </div>

        <div>
          <h3 className="font-semibold">Chapters</h3>
          <ul className="mt-2 space-y-2">
            {subjectId ? (
              subjects.find(s => s.id === subjectId)?.chapters.map(c => (
                <li key={c.id} className={`p-2 rounded cursor-pointer ${chapterId===c.id? 'bg-amber-50' : 'hover:bg-slate-50'}`} onClick={() => selectChapter(c.id)}>
                  {c.title}
                  <div className="text-xs text-slate-400">{c.questions.length} Qs</div>
                </li>
              )) : (<li className="text-slate-400">Select a subject</li>)
            }
          </ul>
        </div>
      </aside>

      <section className="md:col-span-3">
        <div className="flex items-center gap-3 mb-4">
          <input value={query} onChange={e => setQuery(e.target.value)} placeholder="Search questions or answers..." className="flex-1 p-2 border rounded" />
          <label className="inline-flex items-center gap-2">
            <input type="checkbox" checked={showExtraOnly} onChange={e => setShowExtraOnly(e.target.checked)} />
            Extras only
          </label>
        </div>

        <div className="bg-white p-4 rounded shadow-sm">
          {!subjectId && <div className="text-slate-500">Choose a subject to view chapters and questions.</div>}

          {subjectId && !chapterId && <div className="text-slate-500">Choose a chapter to view questions.</div>}

          {subjectId && chapterId && (
            <div>
              <div className="flex justify-between items-center mb-3">
                <h2 className="text-xl font-semibold">Questions</h2>
                <small className="text-slate-500">{filteredQuestions().length} results</small>
              </div>

              <ol className="space-y-4">
                {filteredQuestions().map(q => (
                  <li key={q.id} className="p-3 border rounded">
                    <div className="font-medium">Q: {q.q}</div>
                    <details className="mt-2">
                      <summary className="cursor-pointer text-sm text-indigo-600">View answer</summary>
                      <div className="mt-2 text-sm text-slate-700">{q.a}</div>
                    </details>
                    <div className="text-xs text-slate-400 mt-2">Tags: {q.tags?.join(', ') || '-'}</div>
                  </li>
                ))}
              </ol>
            </div>
          )}
        </div>

        <div className="mt-6 bg-white p-4 rounded shadow-sm">
          <h3 className="font-semibold">Add a question (Admin)</h3>
          <AddQuestionForm subjects={subjects} onAdd={(payload) => addQuestion({...payload, subjectId})} chapterId={chapterId} subjectId={subjectId} />
        </div>

        <div className="mt-6 bg-white p-4 rounded shadow-sm">
          <h3 className="font-semibold">Extra questions (search-ready)</h3>
          <p className="text-sm text-slate-500">Extra / practice questions that are not directly in NCERT but useful for tests.</p>
          <div className="mt-3">
            {renderExtras(data)}
          </div>
        </div>

      </section>
    </main>

    <footer className="p-4 text-center text-sm text-slate-500">Built for NCERT practice — replace SAMPLE_DATA with your full dataset.</footer>
  </div>
</div>

) }

function AddQuestionForm({subjects, onAdd, subjectId, chapterId}){ const [qText, setQText] = useState('') const [aText, setAText] = useState('') const [sId, setSId] = useState(subjectId || (subjects[0] && subjects[0].id)) const [cId, setCId] = useState(chapterId || (subjects[0] && subjects[0].chapters[0] && subjects[0].chapters[0].id)) const [isExtra, setIsExtra] = useState(false)

React.useEffect(()=>{ if(subjectId) setSId(subjectId) if(chapterId) setCId(chapterId) }, [subjectId, chapterId])

const chaptersForSelected = subjects.find(s => s.id === sId)?.chapters || []

function submit(e){ e.preventDefault() if(!qText || !aText) return alert('Add both Q and A') onAdd({subjectId: sId, chapterId: cId, qText, aText, tags: isExtra ? ['extra'] : []}) setQText('') setAText('') alert('Question added (in-memory)') }

return ( <form onSubmit={submit} className="space-y-3"> <div className="grid grid-cols-2 gap-2"> <select value={sId} onChange={e => setSId(e.target.value)} className="p-2 border rounded"> {subjects.map(s => <option key={s.id} value={s.id}>{s.name}</option>)} </select> <select value={cId} onChange={e => setCId(e.target.value)} className="p-2 border rounded"> {chaptersForSelected.map(c => <option key={c.id} value={c.id}>{c.title}</option>)} </select> </div> <input value={qText} onChange={e => setQText(e.target.value)} placeholder="Question text" className="w-full p-2 border rounded" /> <textarea value={aText} onChange={e => setAText(e.target.value)} placeholder="Answer text" className="w-full p-2 border rounded" rows={4} /> <div className="flex items-center gap-3"> <label className="inline-flex items-center gap-2"><input type="checkbox" checked={isExtra} onChange={e => setIsExtra(e.target.checked)} /> Mark as extra</label> <button type="submit" className="px-3 py-2 bg-indigo-600 text-white rounded">Add</button> </div> </form> ) }

function renderExtras(data){ const extras = [] Object.keys(data).forEach(yearKey => { data[yearKey].subjects.forEach(sub => { sub.chapters.forEach(ch => { ch.questions.forEach(q => { if(q.tags?.includes('extra')) extras.push({year: yearKey, subject: sub.name, chapter: ch.title, q, id: q.id}) }) }) }) })

if(extras.length === 0) return <div className="text-slate-400">No extra questions yet.</div> return ( <ul className="space-y-3"> {extras.map(e => ( <li key={e.id} className="p-2 border rounded"> <div className="font-medium">{e.q.q}</div> <div className="text-xs text-slate-500">{e.subject} — {e.chapter}</div> </li> ))} </ul> ) } ncert-qa-portal
