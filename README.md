# songly
import React, { useEffect, useState, useRef } from 'react';

// Pretty single-file React component for a small demo app // Requirements assumed: Tailwind CSS is installed in the project. // Save as src/App.jsx in a create-react-app / Vite React project with Tailwind.

export default function App() { const [items, setItems] = useState([]); const [query, setQuery] = useState(''); const [editing, setEditing] = useState(null); const [theme, setTheme] = useState('rose'); const titleRef = useRef(); const artistRef = useRef(); const lyricsRef = useRef(); const fileRef = useRef();

// load from localStorage useEffect(() => { const raw = localStorage.getItem('songly_items_v1'); const t = localStorage.getItem('songly_theme_v1'); if (raw) setItems(JSON.parse(raw)); if (t) setTheme(t); }, []);

// persist useEffect(() => { localStorage.setItem('songly_items_v1', JSON.stringify(items)); }, [items]);

useEffect(() => { localStorage.setItem('songly_theme_v1', theme); }, [theme]);

function handleAdd(e) { e.preventDefault(); const title = titleRef.current.value.trim(); const artist = artistRef.current.value.trim(); const lyrics = lyricsRef.current.value.trim(); const file = fileRef.current.files[0]; if (!title) return alert('노래 제목을 입력해주세요.');

// if file uploaded, create an object URL (not persisted across sessions if not saved as dataURL)
// For simplicity we convert audio to base64 string (BEWARE: large files => big localStorage)
if (file) {
  const reader = new FileReader();
  reader.onload = () => {
    const dataUrl = reader.result; // base64
    pushNewItem({ title, artist, lyrics, audio: dataUrl });
  };
  reader.readAsDataURL(file);
} else {
  // allow entry with no audio (lyrics-only)
  pushNewItem({ title, artist, lyrics, audio: null });
}

// clear form
titleRef.current.value = '';
artistRef.current.value = '';
lyricsRef.current.value = '';
fileRef.current.value = null;

}

function pushNewItem(payload) { const id = Date.now().toString(); const createdAt = new Date().toISOString(); const newItem = { id, ...payload, createdAt, modified: false }; setItems((s) => [newItem, ...s]); }

function handleDelete(id) { if (!confirm('정말 삭제하시겠습니까?')) return; setItems((s) => s.filter((it) => it.id !== id)); }

function startEdit(item) { setEditing({ ...item }); }

function saveEdit(e) { e.preventDefault(); const updated = { ...editing, modified: true, modifiedAt: new Date().toISOString() }; setItems((s) => s.map((it) => (it.id === updated.id ? updated : it))); setEditing(null); }

function cancelEdit() { setEditing(null); }

function handleAudioChange(e) { const file = e.target.files[0]; if (!file) return; const reader = new FileReader(); reader.onload = () => { setEditing((ed) => ({ ...ed, audio: reader.result })); }; reader.readAsDataURL(file); }

const filtered = items.filter((it) => { const q = query.trim().toLowerCase(); if (!q) return true; return ( it.title.toLowerCase().includes(q) || (it.artist || '').toLowerCase().includes(q) || (it.lyrics || '').toLowerCase().includes(q) ); });

function exportJSON() { const data = JSON.stringify(items, null, 2); const blob = new Blob([data], { type: 'application/json' }); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href = url; a.download = songly-export-${new Date().toISOString()}.json; a.click(); URL.revokeObjectURL(url); }

return ( <div className={min-h-screen p-6 bg-gradient-to-b from-${theme}-50 to-${theme}-100}> <div className="max-w-4xl mx-auto"> <header className="flex items-center justify-between mb-6"> <h1 className="text-3xl font-extrabold">예쁜 가사 & 노래 업로드 — Songly</h1> <div className="flex gap-2 items-center"> <select aria-label="테마" className="rounded-md p-2 border" value={theme} onChange={(e) => setTheme(e.target.value)} > <option value="rose">로즈</option> <option value="emerald">에메랄드</option> <option value="amber">앰버</option> <option value="violet">바이올렛</option> </select> <button className="px-3 py-2 border rounded-md" onClick={exportJSON}> 내보내기 </button> </div> </header>

<form onSubmit={handleAdd} className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
      <div className="md:col-span-2 space-y-2">
        <input ref={titleRef} placeholder="노래 제목" className="w-full p-3 rounded-md border" />
        <input ref={artistRef} placeholder="아티스트 (선택)" className="w-full p-3 rounded-md border" />
        <textarea ref={lyricsRef} placeholder="가사 입력 또는 붙여넣기" rows={4} className="w-full p-3 rounded-md border" />
        <div className="flex items-center gap-3">
          <input ref={fileRef} type="file" accept="audio/*" />
          <button className="ml-auto bg-white border px-4 py-2 rounded-md">추가</button>
        </div>
      </div>

      <div className="flex flex-col gap-2 p-4 rounded-lg border bg-white">
        <h3 className="font-semibold">작게 보는 미리보기</h3>
        <p className="text-sm text-gray-600">업로드한 노래와 가사를 아름답게 모아보세요.</p>
        <div className="mt-2 text-xs text-gray-500">로컬에 저장됩니다. 큰 오디오 파일은 로컬스토리지 용량을 크게 차지합니다.</div>
      </div>
    </form>

    <div className="mb-4 flex items-center gap-3">
      <input
        className="flex-1 p-2 border rounded-md"
        placeholder="검색: 제목/아티스트/가사"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <div className="text-sm text-gray-500">{filtered.length}개의 항목</div>
    </div>

    <main>
      <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
        {filtered.map((it) => (
          <article key={it.id} className={`p-4 rounded-2xl shadow-md bg-white relative`}> 
            <div className="flex justify-between items-start">
              <div>
                <h2 className="text-lg font-semibold">{it.title}</h2>
                <div className="text-sm text-gray-600">{it.artist}</div>
                <div className="mt-2 text-sm whitespace-pre-wrap">{it.lyrics ? (it.lyrics.length>300 ? it.lyrics.slice(0,300)+"..." : it.lyrics) : <em className="text-gray-400">가사 없음</em>}</div>
              </div>
              <div className="flex flex-col items-end gap-2">
                <div className="text-xs text-gray-400">{new Date(it.createdAt).toLocaleString()}</div>
                <div className="flex gap-2">
                  <button className="px-3 py-1 border rounded-md" onClick={() => startEdit(it)}>수정</button>
                  <button className="px-3 py-1 border rounded-md" onClick={() => handleDelete(it.id)}>삭제</button>
                </div>
              </div>
            </div>

            {it.audio && (
              <div className="mt-3">
                <audio controls src={it.audio} className="w-full rounded-md" />
              </div>
            )}

            {it.modified && <div className="absolute top-3 left-3 text-xs px-2 py-1 rounded-full bg-yellow-100 text-yellow-700">수정됨</div>}
          </article>
        ))}

        {filtered.length === 0 && (
          <div className="p-6 text-center text-gray-500">검색 결과가 없습니다. 새 항목을 추가해보세요.</div>
        )}
      </div>
    </main>

    {/* Edit modal (simple) */}
    {editing && (
      <div className="fixed inset-0 bg-black/40 flex items-center justify-center p-4">
        <form onSubmit={saveEdit} className="w-full max-w-2xl bg-white p-6 rounded-2xl">
          <h3 className="text-xl font-bold mb-2">항목 수정</h3>
          <div className="grid grid-cols-1 gap-3">
            <input
              value={editing.title}
              onChange={(e) => setEditing((s) => ({ ...s, title: e.target.value }))}
              className="p-3 border rounded-md"
            />
            <input
              value={editing.artist || ''}
              onChange={(e) => setEditing((s) => ({ ...s, artist: e.target.value }))}
              className="p-3 border rounded-md"
            />
            <textarea
              value={editing.lyrics || ''}
              onChange={(e) => setEditing((s) => ({ ...s, lyrics: e.target.value }))}
              rows={6}
              className="p-3 border rounded-md"
            />
            <div className="flex items-center gap-3">
              <input type="file" accept="audio/*" onChange={handleAudioChange} />
              <div className="ml-auto flex gap-2">
                <button type="button" className="px-4 py-2 border rounded-md" onClick={cancelEdit}>
                  취소
                </button>
                <button className="px-4 py-2 bg-black text-white rounded-md">저장</button>
              </div>
            </div>
          </div>
        </form>
      </div>
    )}

    <footer className="mt-12 text-center text-sm text-gray-500">
      저작권: 가사/음원 업로드 시 저작권을 확인해주세요. 이 데모는 로컬 스토리지를 사용합니다. (프로덕션: 서버/스토리지 서비스 권장)
    </footer>
  </div>
</div>

); }
