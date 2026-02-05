<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EXEIGHTED-REMINDER</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700;800&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <style>
        body { font-family: 'Plus Jakarta Sans', sans-serif; -webkit-tap-highlight-color: transparent; }
        .animate-in { animation: fadeIn 0.3s ease-out forwards; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .hide-scrollbar::-webkit-scrollbar { display: none; }
        .active-tab { color: #acb932; transform: scale(1.1); font-weight: 800; }
        .whitespace-pre-wrap { white-space: pre-wrap; word-break: break-word; }
    </style>
</head>
<body class="bg-slate-50 text-slate-900">
    <div id="root"></div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.2.0/firebase-app.js";
        import { getFirestore, doc, setDoc, onSnapshot, getDoc } from "https://www.gstatic.com/firebasejs/11.2.0/firebase-firestore.js";
        import { getAuth, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.2.0/firebase-auth.js";

        const firebaseConfig = {
            apiKey: "AIzaSyC7Dlgc7nvO5C4TpGK78JLiMgpUPhFOzA0",
            authDomain: "exeighted-site.firebaseapp.com",
            projectId: "exeighted-site",
            storageBucket: "exeighted-site.firebasestorage.app",
            messagingSenderId: "2577177460",
            appId: "1:2577177460:web:58c870583cf4dd25c06a21",
            measurementId: "G-W19QT6WZ8W"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);

        window.FB_DB = db;
        window.FB_AUTH = auth;
        window.FB_TOOLS = { doc, setDoc, onSnapshot, getDoc, signInAnonymously };
        window.APP_ID = "exeighted-v4-final-struct"; 
    </script>

    <script type="text/babel">
        const { useState, useEffect } = React;

        const App = () => {
            const [activeTab, setActiveTab] = useState('reminder');
            const [isAdmin, setIsAdmin] = useState(false);
            const [showLogin, setShowLogin] = useState(false);
            const [password, setPassword] = useState('');
            const [loading, setLoading] = useState(true);
            const [user, setUser] = useState(null);

            const ADMIN_PASS = "Navila139";

            const [classData, setClassData] = useState({
                pembiasaan: "Tadarus & Dhuha",
                mapelBesok: "MTK\nB. Indo\nIPA", 
                tugasBesok: [{ teks: "Bawa LKS IPA hal 20", status: "BIASA" }],
                jurnalHariIni: [{ tanggal: "2024-01-01", mapel: "PABP", materi: "Sejarah Islam", tugas: "Hafalan surah" }],
                waliKelas: "Ibu/Bapak Guru",
                siswa: [
                    { nama: "Abdurrahman", jabatan: "Ketua Kelas" }, { nama: "Nabila", jabatan: "Wakil Ketua" },
                    { nama: "Ahmad", jabatan: "Siswa" }, { nama: "Aisyah", jabatan: "Siswa" },
                    { nama: "Budi", jabatan: "Siswa" }, { nama: "Citra", jabatan: "Siswa" },
                    { nama: "Dedi", jabatan: "Siswa" }, { nama: "Eka", jabatan: "Siswa" },
                    { nama: "Fanny", jabatan: "Siswa" }, { nama: "Gani", jabatan: "Siswa" },
                    { nama: "Hana", jabatan: "Siswa" }, { nama: "Indra", jabatan: "Siswa" },
                    { nama: "Joko", jabatan: "Siswa" }, { nama: "Kiki", jabatan: "Siswa" },
                    { nama: "Lala", jabatan: "Siswa" }, { nama: "Maya", jabatan: "Siswa" },
                    { nama: "Nana", jabatan: "Siswa" }, { nama: "Oki", jabatan: "Siswa" },
                    { nama: "Putri", jabatan: "Siswa" }, { nama: "Qori", jabatan: "Siswa" },
                    { nama: "Rian", jabatan: "Siswa" }, { nama: "Sari", jabatan: "Siswa" },
                    { nama: "Tono", jabatan: "Siswa" }, { nama: "Uci", jabatan: "Siswa" },
                    { nama: "Vina", jabatan: "Siswa" }, { nama: "Wawan", jabatan: "Siswa" },
                    { nama: "Xena", jabatan: "Siswa" }, { nama: "Yanto", jabatan: "Siswa" },
                    { nama: "Zizi", jabatan: "Siswa" }, { nama: "Arif", jabatan: "Siswa" },
                    { nama: "Bella", jabatan: "Siswa" }, { nama: "Chiko", jabatan: "Siswa" },
                ],
                jadwal: {
                    senin: "Upacara\nMTK\nB.Ing",
                    selasa: "Olahraga\nIPA\nSeni",
                    rabu: "B.Indo\nIPS\nAgama",
                    kamis: "PKN\nMTK\nIPA",
                    jumat: "Dhuha\nB.Ing\nBersih-bersih"
                }
            });

            // Urutan hari yang benar
            const hariOrder = ['senin', 'selasa', 'rabu', 'kamis', 'jumat'];

            useEffect(() => {
                const initCloud = async () => {
                    if (!window.FB_TOOLS) { setTimeout(initCloud, 500); return; }
                    try {
                        const { FB_AUTH, FB_DB, FB_TOOLS, APP_ID } = window;
                        const userCred = await FB_TOOLS.signInAnonymously(FB_AUTH);
                        setUser(userCred.user);
                        const docRef = FB_TOOLS.doc(FB_DB, 'artifacts', APP_ID, 'public', 'main_config');
                        const snap = await FB_TOOLS.getDoc(docRef);
                        if (!snap.exists()) await FB_TOOLS.setDoc(docRef, classData);
                        FB_TOOLS.onSnapshot(docRef, (s) => {
                            if (s.exists()) setClassData(s.data());
                            setLoading(false);
                        });
                    } catch (e) { console.error(e); }
                };
                initCloud();
            }, []);

            const handleUpdate = async () => {
                setLoading(true);
                try {
                    const docRef = window.FB_TOOLS.doc(window.FB_DB, 'artifacts', window.APP_ID, 'public', 'main_config');
                    await window.FB_TOOLS.setDoc(docRef, classData);
                    setIsAdmin(false);
                } catch (e) { alert("Error: " + e.message); }
                setLoading(false);
            };

            const autoFillMapel = (hari) => {
                const mapel = classData.jadwal[hari.toLowerCase()] || "";
                setClassData({...classData, mapelBesok: mapel});
            };

            if (loading && !user) return (
                <div className="h-screen flex flex-col items-center justify-center bg-indigo-700 text-white">
                    <div className="w-10 h-10 border-4 border-white/20 border-t-white rounded-full animate-spin mb-4"></div>
                    <p className="font-black text-[9px] tracking-[0.2em] uppercase opacity-60">Connecting Cloud...</p>
                </div>
            );

            return (
                <div className="min-h-screen bg-slate-50 pb-32">
                    <header className="bg-indigo-700 text-white p-6 pt-10 pb-12 rounded-b-[3.5rem] shadow-xl sticky top-0 z-50 flex justify-between items-center">
                        <div>
                            <h1 className="text-2xl font-black italic tracking-tighter">EXEIGHTED REMINDER</h1>
                            <span className="text-[8px] font-bold uppercase tracking-widest bg-white/20 px-2 py-0.5 rounded-full">Kelas X-8</span>
                        </div>
                        <button onClick={() => setShowLogin(true)} className="w-11 h-11 bg-white/10 rounded-2xl flex items-center justify-center text-lg hover:bg-white/20 transition-all active:scale-90">⚙️</button>
                    </header>

                    <main className="max-w-xl mx-auto px-5 py-6">
                        {activeTab === 'reminder' && (
                            <div className="space-y-4 animate-in">
                                <div className="bg-white p-6 rounded-[2.5rem] border border-slate-100 shadow-sm">
                                    <h2 className="text-[10px] font-black text-indigo-600 uppercase mb-4 tracking-widest">✨ Pembiasaan Besok</h2>
                                    <p className="text-xl font-extrabold text-slate-800 leading-tight whitespace-pre-wrap">{classData.pembiasaan}</p>
                                </div>
                                <div className="bg-white p-6 rounded-[2.5rem] border border-slate-100 shadow-sm">
                                    <h2 className="text-[10px] font-black text-emerald-600 uppercase mb-4 tracking-widest">📚 Mapel Besok</h2>
                                    <p className="font-bold text-slate-600 italic leading-relaxed whitespace-pre-wrap">{classData.mapelBesok}</p>
                                </div>
                                <div className="bg-white p-6 rounded-[2.5rem] border border-slate-100 shadow-sm">
                                    <h2 className="text-[10px] font-black text-rose-500 uppercase mb-4 tracking-widest">📌 PR/ULANGAN BESOK</h2>
                                    <div className="space-y-2">
                                        {classData.tugasBesok.map((t, i) => (
                                            <div key={i} className={`p-4 rounded-2xl flex gap-3 items-center ${t.status === 'ULANGAN' ? 'bg-rose-50 border border-rose-100' : t.status === 'PR' ? 'bg-indigo-50 border border-indigo-100' : 'bg-slate-50'}`}>
                                                <div className={`w-2 h-2 rounded-full ${t.status === 'ULANGAN' ? 'bg-rose-500' : t.status === 'PR' ? 'bg-indigo-500' : 'bg-slate-300'}`}></div>
                                                <div className="flex flex-col">
                                                    <span className={`text-[7px] font-black uppercase tracking-widest mb-0.5 ${t.status === 'ULANGAN' ? 'text-rose-500' : t.status === 'PR' ? 'text-indigo-500' : 'text-slate-400'}`}>{t.status || 'BIASA'}</span>
                                                    <span className={`text-sm font-bold whitespace-pre-wrap ${t.status === 'ULANGAN' ? 'text-rose-700' : t.status === 'PR' ? 'text-indigo-700' : 'text-slate-600'}`}>{t.teks}</span>
                                                </div>
                                            </div>
                                        ))}
                                        {classData.tugasBesok.length === 0 && <p className="text-center py-4 text-slate-300 italic text-xs">Tidak ada agenda besok</p>}
                                    </div>
                                </div>
                            </div>
                        )}

                        {activeTab === 'jurnal' && (
                            <div className="space-y-4 animate-in">
                                <div className="bg-indigo-600 p-6 rounded-[2.5rem] shadow-lg mb-6">
                                    <h2 className="text-[10px] font-black text-indigo-200 uppercase tracking-[0.2em] mb-1">----</h2>
                                    <p className="text-white text-lg font-bold italic">JURNAL KELAS</p>
                                </div>
                                {classData.jurnalHariIni.map((j, i) => (
                                    <div key={i} className="bg-white p-6 rounded-[2.5rem] border border-slate-100 shadow-sm">
                                        <div className="flex justify-between items-center mb-4">
                                            <span className="text-[10px] font-black bg-indigo-50 text-indigo-600 px-3 py-1 rounded-full uppercase tracking-widest">{j.mapel}</span>
                                            <span className="text-[10px] font-bold text-slate-400">{j.tanggal}</span>
                                        </div>
                                        <div className="space-y-3">
                                            <div>
                                                <h4 className="text-[8px] font-black text-slate-400 uppercase tracking-widest mb-1">Materi</h4>
                                                <p className="font-bold text-slate-800 text-sm leading-tight whitespace-pre-wrap">{j.materi}</p>
                                            </div>
                                            {j.tugas && (
                                                <div className="pt-2 border-t border-slate-50">
                                                    <h4 className="text-[8px] font-black text-rose-400 uppercase tracking-widest mb-1">Tugas</h4>
                                                    <p className="font-bold text-rose-600 text-[11px] italic whitespace-pre-wrap">📝 {j.tugas}</p>
                                                </div>
                                            )}
                                        </div>
                                    </div>
                                ))}
                            </div>
                        )}

                        {activeTab === 'info' && (
                            <div className="space-y-5 animate-in pb-10">
                                <div className="bg-white p-7 rounded-[2.5rem] border border-slate-100 shadow-sm">
                                    <div className="flex items-center gap-4 mb-6">
                                        <div className="w-14 h-14 bg-indigo-100 rounded-3xl flex items-center justify-center text-2xl">🎓</div>
                                        <div>
                                            <h3 className="text-[10px] font-black text-slate-400 uppercase tracking-widest">Wali Kelas</h3>
                                            <p className="font-black text-indigo-900">{classData.waliKelas}</p>
                                        </div>
                                    </div>
                                    <h3 className="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-4">🗓️ Jadwal Pelajaran</h3>
                                    <div className="grid grid-cols-1 gap-2">
                                        {hariOrder.map((hari) => (
                                            <div key={hari} className="flex bg-slate-50 p-4 rounded-2xl items-start gap-4">
                                                <span className="w-16 font-black text-[9px] uppercase text-indigo-600 mt-1">{hari}</span>
                                                <span className="text-xs font-bold text-slate-600 whitespace-pre-wrap flex-1">{classData.jadwal[hari]}</span>
                                            </div>
                                        ))}
                                    </div>
                                </div>
                                <div className="bg-white p-7 rounded-[2.5rem] border border-slate-100 shadow-sm">
                                    <h3 className="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-4">👥 Struktur Kelas</h3>
                                    <div className="space-y-2">
                                        {classData.siswa.map((s, i) => (
                                            <div key={i} className="flex justify-between items-center p-4 bg-slate-50 rounded-2xl">
                                                <span className="font-bold text-slate-700 text-sm">{s.nama}</span>
                                                <span className="text-[8px] px-3 py-1 bg-white text-indigo-600 rounded-lg font-black uppercase shadow-sm italic">{s.jabatan}</span>
                                            </div>
                                        ))}
                                    </div>
                                </div>
                            </div>
                        )}
                    </main>

                    <nav className="fixed bottom-8 left-8 right-8 bg-white/95 backdrop-blur-xl border border-slate-200 h-24 rounded-[3rem] flex justify-around items-center z-[60] shadow-2xl max-w-lg mx-auto">
                        <button onClick={() => setActiveTab('reminder')} className={`flex flex-col items-center gap-1.5 transition-all ${activeTab === 'reminder' ? 'active-tab' : 'text-slate-300'}`}>
                            <span className="text-2xl">🔔</span>
                            <span className="text-[8px] font-black tracking-widest uppercase">Remind</span>
                        </button>
                        <button onClick={() => setActiveTab('jurnal')} className={`flex flex-col items-center gap-1.5 transition-all ${activeTab === 'jurnal' ? 'active-tab' : 'text-slate-300'}`}>
                            <span className="text-2xl">📖</span>
                            <span className="text-[8px] font-black tracking-widest uppercase">Jurnal</span>
                        </button>
                        <button onClick={() => setActiveTab('info')} className={`flex flex-col items-center gap-1.5 transition-all ${activeTab === 'info' ? 'active-tab' : 'text-slate-300'}`}>
                            <span className="text-2xl">ℹ️</span>
                            <span className="text-[8px] font-black tracking-widest uppercase">Info</span>
                        </button>
                    </nav>

                    {isAdmin && (
                        <div className="fixed inset-0 bg-white z-[100] p-6 overflow-y-auto hide-scrollbar animate-in">
                            <div className="max-w-xl mx-auto">
                                <div className="flex justify-between items-center mb-8 sticky top-0 bg-white py-4 z-10 border-b border-slate-50">
                                    <h2 className="text-xl font-black italic text-indigo-700 uppercase">Editor Database</h2>
                                    <button onClick={() => setIsAdmin(false)} className="px-4 py-2 bg-slate-100 rounded-xl font-black text-[9px]">CLOSE</button>
                                </div>

                                <div className="space-y-12">
                                    <section className="space-y-4">
                                        <h3 className="text-indigo-600 font-black italic text-sm underline">SET REMINDER</h3>
                                        <textarea className="w-full p-4 bg-slate-50 rounded-2xl font-bold border border-slate-100 min-h-[80px]" placeholder="Pembiasaan" value={classData.pembiasaan} onChange={(e) => setClassData({...classData, pembiasaan: e.target.value})} />
                                        
                                        <h4 className="text-[10px] font-bold text-slate-400 uppercase">Pilih Hari Untuk Mapel Besok</h4>
                                        <div className="flex flex-wrap gap-2 mb-4">
                                            {hariOrder.map((hari) => (
                                                <button 
                                                    key={hari} 
                                                    onClick={() => autoFillMapel(hari)}
                                                    className="px-4 py-2 bg-indigo-50 hover:bg-indigo-600 hover:text-white text-indigo-600 rounded-xl text-[10px] font-black transition-all uppercase"
                                                >
                                                    {hari}
                                                </button>
                                            ))}
                                        </div>
                                        <textarea className="w-full p-4 bg-slate-50 rounded-2xl font-bold border border-slate-100 min-h-[120px]" placeholder="Mapel Besok" value={classData.mapelBesok} onChange={(e) => setClassData({...classData, mapelBesok: e.target.value})} />
                                        
                                        <button onClick={() => setClassData({...classData, tugasBesok: [...classData.tugasBesok, {teks: "", status: "BIASA"}]})} className="w-full p-3 bg-rose-50 text-rose-600 rounded-2xl text-[10px] font-black">+ TAMBAH PR/ULANGAN BESOK</button>
                                        {classData.tugasBesok.map((t, i) => (
                                            <div key={i} className="p-4 bg-slate-50 rounded-2xl space-y-2 border border-slate-100">
                                                <textarea className="w-full p-3 rounded-xl bg-white font-bold text-sm min-h-[60px]" placeholder="Isi tugas/ulangan" value={t.teks} onChange={(e) => {
                                                    const nt = [...classData.tugasBesok]; nt[i].teks = e.target.value;
                                                    setClassData({...classData, tugasBesok: nt});
                                                }} />
                                                <div className="flex gap-2">
                                                    {['BIASA', 'PR', 'ULANGAN'].map((status) => (
                                                        <button 
                                                            key={status}
                                                            onClick={() => {
                                                                const nt = [...classData.tugasBesok]; nt[i].status = status;
                                                                setClassData({...classData, tugasBesok: nt});
                                                            }} 
                                                            className={`flex-1 p-2 rounded-lg text-[8px] font-black transition-all ${t.status === status ? (status === 'ULANGAN' ? 'bg-rose-500 text-white' : status === 'PR' ? 'bg-indigo-500 text-white' : 'bg-slate-700 text-white') : 'bg-slate-200 text-slate-500'}`}
                                                        >
                                                            {status}
                                                        </button>
                                                    ))}
                                                    <button onClick={() => {
                                                        const nt = classData.tugasBesok.filter((_, idx) => idx !== i);
                                                        setClassData({...classData, tugasBesok: nt});
                                                    }} className="p-2 bg-white border border-slate-200 text-rose-500 rounded-lg text-[8px] font-black">HAPUS</button>
                                                </div>
                                            </div>
                                        ))}
                                    </section>

                                    <section className="space-y-4">
                                        <h3 className="text-indigo-600 font-black italic text-sm underline">SET JURNAL HARI INI</h3>
                                        <button onClick={() => setClassData({...classData, jurnalHariIni: [{tanggal: new Date().toISOString().split('T')[0], mapel: "", materi: "", tugas: ""}, ...classData.jurnalHariIni]})} className="w-full p-3 bg-indigo-50 text-indigo-600 rounded-2xl text-[10px] font-black">+ TAMBAH JURNAL</button>
                                        {classData.jurnalHariIni.map((j, i) => (
                                            <div key={i} className="p-5 bg-slate-50 rounded-3xl space-y-3 border border-slate-100">
                                                <div className="grid grid-cols-2 gap-2">
                                                    <input type="date" className="w-full p-3 rounded-xl bg-white text-xs font-bold" value={j.tanggal} onChange={(e) => {
                                                        const nj = [...classData.jurnalHariIni]; nj[i].tanggal = e.target.value;
                                                        setClassData({...classData, jurnalHariIni: nj});
                                                    }} />
                                                    <input className="w-full p-3 rounded-xl bg-white font-bold text-xs" placeholder="Mapel" value={j.mapel} onChange={(e) => {
                                                        const nj = [...classData.jurnalHariIni]; nj[i].mapel = e.target.value;
                                                        setClassData({...classData, jurnalHariIni: nj});
                                                    }} />
                                                </div>
                                                <textarea className="w-full p-3 rounded-xl bg-white text-sm font-medium" placeholder="Materi" value={j.materi} onChange={(e) => {
                                                    const nj = [...classData.jurnalHariIni]; nj[i].materi = e.target.value;
                                                    setClassData({...classData, jurnalHariIni: nj});
                                                }} />
                                                <textarea className="w-full p-3 rounded-xl bg-white text-xs italic" placeholder="Tugas" value={j.tugas} onChange={(e) => {
                                                    const nj = [...classData.jurnalHariIni]; nj[i].tugas = e.target.value;
                                                    setClassData({...classData, jurnalHariIni: nj});
                                                }} />
                                                <button onClick={() => {
                                                    const nj = classData.jurnalHariIni.filter((_, idx) => idx !== i);
                                                    setClassData({...classData, jurnalHariIni: nj});
                                                }} className="w-full text-[9px] font-black text-rose-400">HAPUS JURNAL INI</button>
                                            </div>
                                        ))}
                                    </section>

                                    <section className="space-y-4">
                                        <h3 className="text-indigo-600 font-black italic text-sm underline">SET INFO KELAS</h3>
                                        <input className="w-full p-4 bg-slate-50 rounded-2xl font-bold border border-slate-100" placeholder="Wali Kelas" value={classData.waliKelas} onChange={(e) => setClassData({...classData, waliKelas: e.target.value})} />
                                        
                                        <h4 className="text-[10px] font-bold text-slate-400 uppercase mt-4">Edit Jadwal (Enter untuk baris baru)</h4>
                                        <div className="space-y-2">
                                            {hariOrder.map((hari) => (
                                                <div key={hari} className="flex flex-col gap-1">
                                                    <span className="text-[9px] font-black uppercase text-indigo-600 ml-2">{hari}</span>
                                                    <textarea className="w-full p-3 bg-slate-50 rounded-xl text-xs outline-none focus:bg-white border border-slate-100 min-h-[80px]" value={classData.jadwal[hari]} onChange={(e) => {
                                                        const nj = {...classData.jadwal}; nj[hari] = e.target.value;
                                                        setClassData({...classData, jadwal: nj});
                                                    }} />
                                                </div>
                                            ))}
                                        </div>
                                        
                                        <h4 className="text-[10px] font-bold text-slate-400 uppercase mt-8">Edit Siswa</h4>
                                        <div className="grid grid-cols-1 gap-2">
                                            {classData.siswa.map((s, i) => (
                                                <div key={i} className="flex gap-2 p-2 bg-slate-50 rounded-xl">
                                                    <input className="flex-1 p-2 bg-white rounded-lg text-xs font-bold" value={s.nama} onChange={(e) => {
                                                        const ns = [...classData.siswa]; ns[i].nama = e.target.value;
                                                        setClassData({...classData, siswa: ns});
                                                    }} />
                                                    <input className="w-24 p-2 bg-white rounded-lg text-[10px] uppercase font-black" value={s.jabatan} onChange={(e) => {
                                                        const ns = [...classData.siswa]; ns[i].jabatan = e.target.value;
                                                        setClassData({...classData, siswa: ns});
                                                    }} />
                                                </div>
                                            ))}
                                            <button onClick={() => setClassData({...classData, siswa: [...classData.siswa, {nama: "Nama Baru", jabatan: "Siswa"}]})} className="p-3 bg-indigo-50 text-indigo-600 rounded-xl text-[9px] font-black">+ TAMBAH SISWA</button>
                                        </div>
                                    </section>

                                    <div className="pt-10 pb-20">
                                        <button onClick={handleUpdate} className="w-full p-6 bg-indigo-600 text-white rounded-[2.5rem] font-black shadow-xl flex items-center justify-center gap-3 active:scale-95 transition-all">
                                            {loading ? <div className="w-5 h-5 border-2 border-white/20 border-t-white animate-spin rounded-full"></div> : "🚀 UPDATE DATABASE SEKARANG"}
                                        </button>
                                        <p className="text-center text-[10px] text-slate-300 font-bold mt-4 uppercase tracking-[0.3em]">EXEIGHTED Cloud Management</p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    )}

                    {showLogin && (
                        <div className="fixed inset-0 bg-slate-900/80 backdrop-blur-md z-[110] flex items-center justify-center p-6">
                            <div className="bg-white w-full max-w-sm rounded-[3.5rem] p-10 text-center shadow-2xl animate-in">
                                <h3 className="text-xl font-black mb-6 italic text-indigo-700 uppercase tracking-tighter">Admin Akses</h3>
                                <input 
                                    type="password" 
                                    placeholder="Password" 
                                    className="w-full p-5 bg-slate-100 rounded-[2rem] mb-4 text-center font-bold outline-none border-2 border-transparent focus:border-indigo-400" 
                                    value={password} 
                                    onChange={(e) => setPassword(e.target.value)} 
                                />
                                <button onClick={() => { if(password === ADMIN_PASS){setIsAdmin(true); setShowLogin(false); setPassword('');} else alert("Akses Ditolak!"); }} className="w-full p-5 bg-indigo-600 text-white rounded-[2rem] font-black shadow-lg hover:bg-indigo-700">Login</button>
                                <button onClick={() => setShowLogin(false)} className="mt-5 text-slate-400 text-[9px] font-black uppercase tracking-widest block w-full">batal</button>
                            </div>
                        </div>
                    )}
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
