import React, { useState, useEffect, useMemo } from 'react';
import { 
  BookOpen, 
  Clock, 
  Target, 
  TrendingUp, 
  Award, 
  BarChart2, 
  CheckCircle, 
  XCircle, 
  Play, 
  Home, 
  GraduationCap,
  ChevronRight,
  AlertCircle,
  Save,
  RefreshCw,
  Sparkles,
  MessageSquare,
  Loader2
} from 'lucide-react';

// --- API CONFIGURATION ---
const apiKey = ""; // API Key will be injected by the environment

// --- DATA MOCKUP (SIMULASI DATABASE) ---

const UNIVERSITIES = [
  { id: 'ui', name: 'Universitas Indonesia (UI)', location: 'Depok', programs: [
    { name: 'Pendidikan Dokter', passingGrade: 720, type: 'Saintek' },
    { name: 'Ilmu Komputer', passingGrade: 700, type: 'Saintek' },
    { name: 'Akuntansi', passingGrade: 680, type: 'Soshum' },
    { name: 'Hukum', passingGrade: 670, type: 'Soshum' }
  ]},
  { id: 'itb', name: 'Institut Teknologi Bandung (ITB)', location: 'Bandung', programs: [
    { name: 'STEI - Komputasi', passingGrade: 715, type: 'Saintek' },
    { name: 'FTTM', passingGrade: 710, type: 'Saintek' },
    { name: 'SBM', passingGrade: 690, type: 'Soshum' }
  ]},
  { id: 'ugm', name: 'Universitas Gadjah Mada (UGM)', location: 'Yogyakarta', programs: [
    { name: 'Kedokteran', passingGrade: 710, type: 'Saintek' },
    { name: 'Teknik Sipil', passingGrade: 650, type: 'Saintek' },
    { name: 'Psikologi', passingGrade: 660, type: 'Soshum' }
  ]},
  { id: 'its', name: 'Institut Teknologi Sepuluh Nopember (ITS)', location: 'Surabaya', programs: [
    { name: 'Teknik Informatika', passingGrade: 680, type: 'Saintek' },
    { name: 'Sistem Informasi', passingGrade: 660, type: 'Saintek' }
  ]}
];

// Bank Soal Diperluas (Sekarang ada 20 Soal Unik)
const QUESTION_BANK = [
  // --- PENALARAN UMUM & KOGNITIF ---
  {
    id: 1,
    type: 'Potensi Kognitif',
    question: 'Jika semua dokter adalah orang pintar, dan sebagian orang pintar adalah seniman. Maka simpulan yang paling tepat adalah...',
    options: ['Sebagian dokter adalah seniman', 'Semua seniman adalah dokter', 'Sebagian seniman adalah dokter', 'Sebagian orang pintar yang bukan dokter adalah seniman', 'Tidak dapat ditarik kesimpulan'],
    correct: 3,
    explanation: 'Karena himpunan dokter ada di dalam himpunan orang pintar, dan himpunan seniman beririsan dengan orang pintar, belum tentu irisan seniman menyentuh himpunan dokter.'
  },
  {
    id: 6,
    type: 'Potensi Kognitif',
    question: 'Pola bilangan: 2, 5, 10, 17, 26, ... Bilangan selanjutnya adalah?',
    options: ['35', '37', '36', '40', '50'],
    correct: 1,
    explanation: 'Pola bertingkat. Selisihnya: +3, +5, +7, +9. Maka selanjutnya +11. 26 + 11 = 37.'
  },
  {
    id: 7,
    type: 'Potensi Kognitif',
    question: 'Semua mamalia bernapas dengan paru-paru. Ikan paus bernapas dengan paru-paru. Kesimpulannya?',
    options: ['Ikan paus adalah mamalia', 'Semua yang bernapas dengan paru-paru adalah mamalia', 'Ikan paus bukan ikan', 'Belum tentu ikan paus adalah mamalia', 'Semua ikan bernapas dengan paru-paru'],
    correct: 3,
    explanation: 'Logika silogisme. Premis 1: A -> B. Premis 2: C adalah B. Tidak bisa disimpulkan C adalah A. Bisa saja ada hewan lain (reptil) yang juga bernapas dengan paru-paru.'
  },
  
  // --- PENALARAN MATEMATIKA ---
  {
    id: 2,
    type: 'Penalaran Matematika',
    question: 'Toko A diskon 20% + 10%. Toko B diskon langsung 30%. Mana yang lebih murah untuk barang seharga 100rb?',
    options: ['Toko A', 'Toko B', 'Sama saja', 'Tergantung pajaknya', 'Tidak bisa ditentukan'],
    correct: 1,
    explanation: 'Toko A: 100rb diskon 20% jadi 80rb. Lalu 80rb diskon 10% jadi 72rb. Toko B: 100rb diskon 30% jadi 70rb. Toko B lebih murah.'
  },
  {
    id: 8,
    type: 'Penalaran Matematika',
    question: 'Sebuah bak mandi bervolume 300 liter bocor. Debit kebocoran 2 liter/menit. Berapa jam bak tersebut akan habis?',
    options: ['1,5 jam', '2 jam', '2,5 jam', '3 jam', '150 menit'],
    correct: 2,
    explanation: 'Waktu = Volume / Debit = 300 / 2 = 150 menit. 150 menit / 60 = 2,5 jam.'
  },
  {
    id: 9,
    type: 'Penalaran Matematika',
    question: 'Rata-rata nilai 5 siswa adalah 80. Jika ditambah nilai Andi, rata-ratanya menjadi 81. Berapa nilai Andi?',
    options: ['81', '85', '86', '90', '95'],
    correct: 2,
    explanation: 'Total awal = 5 x 80 = 400. Total baru = 6 x 81 = 486. Nilai Andi = 486 - 400 = 86.'
  },

  // --- LITERASI INDONESIA ---
  {
    id: 3,
    type: 'Literasi Bahasa Indonesia',
    question: 'Manakah penulisan kata serapan yang benar menurut PUEBI?',
    options: ['Analisa, Resiko, Praktek', 'Analisis, Risiko, Praktik', 'Analisa, Risiko, Praktik', 'Analisis, Resiko, Praktek', 'Analisis, Resiko, Praktik'],
    correct: 1,
    explanation: 'Baku: Analisis (bukan analisa), Risiko (bukan resiko), Praktik (bukan praktek).'
  },
  {
    id: 10,
    type: 'Literasi Bahasa Indonesia',
    question: 'Kalimat manakah yang efektif?',
    options: ['Para hadirin dimohon berdiri', 'Bagi semua peserta diharapkan hadir', 'Dia masuk ke dalam ruangan', 'Hadirin dimohon berdiri', 'Kepada Bapak Kepala Sekolah kami persilakan'],
    correct: 3,
    explanation: '"Para hadirin" pleonasme (para = jamak, hadirin = jamak). "Masuk ke dalam" mubazir. "Bagi semua" subjek tidak jelas. Yang benar: "Hadirin dimohon berdiri".'
  },
  {
    id: 11,
    type: 'Literasi Bahasa Indonesia',
    question: 'Inti kalimat: "Pemerintah yang baru terbentuk segera menyusun program kerja 100 hari."',
    options: ['Pemerintah menyusun', 'Pemerintah menyusun program', 'Pemerintah menyusun program kerja', 'Pemerintah baru menyusun', 'Pemerintah segera menyusun'],
    correct: 1,
    explanation: 'Subjek: Pemerintah (yang baru terbentuk = perluasan). Predikat: menyusun. Objek: program kerja (100 hari = perluasan). Inti: Pemerintah menyusun.'
  },

  // --- LITERASI INGGRIS ---
  {
    id: 4,
    type: 'Literasi Bahasa Inggris',
    question: 'The phrase "break the ice" typically means to...',
    options: ['Destroy an ice sculpture', 'Make people feel more comfortable in a social setting', 'End a relationship suddenly', 'Start a conflict', 'Cool down a heated argument'],
    correct: 1,
    explanation: 'Idiom "break the ice" berarti mencairkan suasana agar orang merasa nyaman.'
  },
  {
    id: 12,
    type: 'Literasi Bahasa Inggris',
    question: 'She _____ to the cinema every weekend.',
    options: ['Go', 'Goes', 'Going', 'Went', 'Gone'],
    correct: 1,
    explanation: 'Simple Present Tense karena ada "every weekend". Subjek "She" (singular) maka verb + es = Goes.'
  },
  {
    id: 13,
    type: 'Literasi Bahasa Inggris',
    question: 'Synonym of "Obsolete" is...',
    options: ['Modern', 'Outdated', 'Sophisticated', 'New', 'Current'],
    correct: 1,
    explanation: 'Obsolete artinya usang atau sudah tidak dipakai lagi. Sinonim paling tepat adalah Outdated.'
  },

  // --- KUANTITATIF ---
  {
    id: 5,
    type: 'Pengetahuan Kuantitatif',
    question: 'Jika x = 3 dan y = -2, berapakah nilai dari x² - 2xy + y²?',
    options: ['1', '5', '13', '25', '36'],
    correct: 3,
    explanation: 'Rumus (x-y)². (3 - (-2))² = (5)² = 25.'
  },
  {
    id: 14,
    type: 'Pengetahuan Kuantitatif',
    question: 'Bilangan berikut yang habis dibagi 3 dan 4 adalah...',
    options: ['1432', '2024', '1524', '1000', '1234'],
    correct: 2,
    explanation: 'Habis dibagi 4: dua digit terakhir habis dibagi 4 (24 habis). Habis dibagi 3: jumlah digit habis dibagi 3 (1+5+2+4=12 habis). Jawaban: 1524.'
  },
  {
    id: 15,
    type: 'Pengetahuan Kuantitatif',
    question: 'Jika 2^x = 32 dan 3^y = 27, maka x + y = ...',
    options: ['7', '8', '9', '5', '3'],
    correct: 1,
    explanation: '2^x = 32 -> x = 5. 3^y = 27 -> y = 3. x + y = 5 + 3 = 8.'
  },
  {
    id: 16,
    type: 'Pengetahuan Kuantitatif',
    question: 'Sudut terkecil yang dibentuk jarum jam pada pukul 04.00 adalah...',
    options: ['90 derajat', '100 derajat', '120 derajat', '150 derajat', '130 derajat'],
    correct: 2,
    explanation: 'Setiap 1 jam = 30 derajat. Pukul 4 berarti 4 x 30 = 120 derajat.'
  },
  {
    id: 17,
    type: 'Pengetahuan Umum',
    question: 'Ibukota baru Indonesia terletak di provinsi...',
    options: ['Kalimantan Barat', 'Kalimantan Tengah', 'Kalimantan Timur', 'Kalimantan Selatan', 'Sulawesi Selatan'],
    correct: 2,
    explanation: 'IKN (Ibu Kota Nusantara) terletak di Kalimantan Timur.'
  },
  {
    id: 18,
    type: 'Potensi Kognitif',
    question: 'Anton lebih tinggi dari Budi. Caca lebih pendek dari Budi. Siapa yang paling tinggi?',
    options: ['Anton', 'Budi', 'Caca', 'Tidak bisa ditentukan', 'Anton dan Budi'],
    correct: 0,
    explanation: 'Anton > Budi. Budi > Caca. Maka urutannya: Anton > Budi > Caca. Paling tinggi Anton.'
  },
  {
    id: 19,
    type: 'Penalaran Matematika',
    question: 'Jarak A ke B 60km. Ditempuh dengan kecepatan 40km/jam. Berapa menit waktu tempuh?',
    options: ['60 menit', '80 menit', '90 menit', '100 menit', '120 menit'],
    correct: 2,
    explanation: 'Waktu = Jarak/Kecepatan = 60/40 = 1.5 jam. 1.5 jam = 90 menit.'
  },
  {
    id: 20,
    type: 'Literasi Bahasa Inggris',
    question: 'Which sentence is Passive Voice?',
    options: ['I eat an apple', 'An apple is eaten by me', 'I am eating an apple', 'She eats apples', 'They ate apples'],
    correct: 1,
    explanation: 'Passive voice cirinya to be + V3 (is eaten).'
  }
];

// Paket Tryout
const PACKAGES = [
  { id: 'pkt-001', title: 'Paket UTBK Asli 2023 (Simulasi)', count: 20, time: 30, description: 'Soal campuran berbagai subtes.' },
  { id: 'pkt-002', title: 'Paket Prediksi SNBT 2025', count: 40, time: 60, description: 'Simulasi panjang dengan soal HOTS.' },
  { id: 'pkt-003', title: 'Drilling TPS - Potensi Kognitif', count: 10, time: 15, description: 'Fokus pada kemampuan logika dasar.' },
];

// --- COMPONENTS ---

const Card = ({ children, className = "" }) => (
  <div className={`bg-white rounded-xl shadow-sm border border-slate-100 p-6 ${className}`}>
    {children}
  </div>
);

const Button = ({ children, onClick, variant = 'primary', className = "", disabled = false }) => {
  const baseStyle = "px-4 py-2 rounded-lg font-medium transition-all duration-200 flex items-center justify-center gap-2";
  const variants = {
    primary: "bg-indigo-600 text-white hover:bg-indigo-700 shadow-indigo-200 shadow-lg",
    secondary: "bg-white text-slate-700 border border-slate-200 hover:bg-slate-50",
    outline: "bg-transparent text-indigo-600 border border-indigo-600 hover:bg-indigo-50",
    danger: "bg-rose-500 text-white hover:bg-rose-600",
    success: "bg-emerald-500 text-white hover:bg-emerald-600",
    ai: "bg-gradient-to-r from-violet-600 to-fuchsia-600 text-white hover:from-violet-700 hover:to-fuchsia-700 shadow-lg shadow-fuchsia-200"
  };
  
  return (
    <button 
      onClick={onClick} 
      disabled={disabled}
      className={`${baseStyle} ${variants[variant]} ${disabled ? 'opacity-50 cursor-not-allowed' : ''} ${className}`}
    >
      {children}
    </button>
  );
};

const ProgressBar = ({ value, max, color = "bg-indigo-500" }) => {
  const percentage = Math.min(100, Math.max(0, (value / max) * 100));
  return (
    <div className="w-full h-3 bg-slate-100 rounded-full overflow-hidden">
      <div 
        className={`h-full ${color} transition-all duration-500 ease-out`} 
        style={{ width: `${percentage}%` }}
      />
    </div>
  );
};

const MarkdownRenderer = ({ text }) => {
  if (!text) return null;
  const lines = text.split('\n');
  return (
    <div className="space-y-2 text-slate-700 text-sm leading-relaxed">
      {lines.map((line, i) => {
        if (line.trim() === '') return <div key={i} className="h-2"></div>;
        const parts = line.split(/(\*\*.*?\*\*)/g);
        return (
          <p key={i}>
            {parts.map((part, j) => {
              if (part.startsWith('**') && part.endsWith('**')) {
                return <strong key={j} className="text-slate-900">{part.slice(2, -2)}</strong>;
              }
              return part;
            })}
          </p>
        );
      })}
    </div>
  );
};

// --- MAIN APP ---

export default function UtbkApp() {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [userData, setUserData] = useState({
    name: 'Pejuang PTN',
    targetUniv: 'ui',
    targetProdi: 'Ilmu Komputer',
    dailyTarget: 20,
    dailyProgress: 0,
    streak: 3,
    studyPlan: null,
    history: []
  });

  const [activeExam, setActiveExam] = useState(null);
  const [examState, setExamState] = useState({
    currentQuestionIndex: 0,
    answers: {},
    timeLeft: 0,
    isFinished: false,
    score: 0
  });

  const [aiLoading, setAiLoading] = useState(false);
  const [aiExplanation, setAiExplanation] = useState(null);

  useEffect(() => {
    const savedData = localStorage.getItem('utbkProData');
    if (savedData) {
      const parsed = JSON.parse(savedData);
      setUserData({ ...parsed, studyPlan: parsed.studyPlan || null });
    }
  }, []);

  useEffect(() => {
    localStorage.setItem('utbkProData', JSON.stringify(userData));
  }, [userData]);

  useEffect(() => {
    let timer;
    if (activeExam && !examState.isFinished && examState.timeLeft > 0) {
      timer = setInterval(() => {
        setExamState(prev => {
          if (prev.timeLeft <= 1) {
            finishExam(); 
            return { ...prev, timeLeft: 0 };
          }
          return { ...prev, timeLeft: prev.timeLeft - 1 };
        });
      }, 1000);
    }
    return () => clearInterval(timer);
  }, [activeExam, examState.isFinished]);

  const callGemini = async (prompt) => {
    try {
      const response = await fetch(
        `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`,
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] }),
        }
      );
      const data = await response.json();
      return data.candidates[0].content.parts[0].text;
    } catch (error) {
      console.error("Gemini API Error:", error);
      throw error;
    }
  };

  const generateStudyPlan = async () => {
    setAiLoading(true);
    const univName = UNIVERSITIES.find(u => u.id === userData.targetUniv)?.name || userData.targetUniv;
    
    const prompt = `Saya siswa SMA persiapan UTBK/SNBT. Tujuan: ${univName}, Jurusan ${userData.targetProdi}.
    Buatkan rencana belajar 1 minggu yang santai tapi efektif.
    Format: Hari 1-7, bold (**Materi**), tips singkat. Bahasa gaul/santai. Jangan pakai markdown heading.`;

    try {
      const result = await callGemini(prompt);
      setUserData(prev => ({ ...prev, studyPlan: result }));
    } catch (error) {
      alert("Gagal koneksi ke AI.");
    } finally {
      setAiLoading(false);
    }
  };

  const getAiExplanation = async (question) => {
    if (aiExplanation?.id === question.id && aiExplanation?.text) return;
    setAiExplanation({ id: question.id, loading: true, text: null });
    
    const prompt = `Jelaskan jawaban soal UTBK ini dengan asik dan mudah dimengerti:
    "${question.question}"
    Jawaban benar: ${question.options[question.correct]}.
    Berikan tips/trik cepat jika ada.`;

    try {
      const result = await callGemini(prompt);
      setAiExplanation({ id: question.id, loading: false, text: result });
    } catch (error) {
      setAiExplanation({ id: question.id, loading: false, text: "Gagal memuat penjelasan AI." });
    }
  };

  // --- LOGIKA BARU PENGACAKAN SOAL ---
  const startExam = (pkg) => {
    // Fungsi Fisher-Yates Shuffle untuk mengacak array
    const shuffleArray = (array) => {
      const newArr = [...array];
      for (let i = newArr.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [newArr[i], newArr[j]] = [newArr[j], newArr[i]];
      }
      return newArr;
    };

    let selectedQuestions = [];
    
    // Jika jumlah soal di paket lebih sedikit atau sama dengan bank soal
    if (pkg.count <= QUESTION_BANK.length) {
      // Acak seluruh bank soal, lalu ambil n pertama
      const shuffled = shuffleArray(QUESTION_BANK);
      selectedQuestions = shuffled.slice(0, pkg.count);
    } else {
      // Jika paket butuh lebih banyak soal dari yang tersedia di bank (misal bank 20, paket 40)
      // Kita harus mengulang, tapi tetap diacak per batch agar tidak monoton
      let remaining = pkg.count;
      while (remaining > 0) {
        const shuffled = shuffleArray(QUESTION_BANK);
        const take = Math.min(remaining, shuffled.length);
        selectedQuestions = [...selectedQuestions, ...shuffled.slice(0, take)];
        remaining -= take;
      }
    }

    // Beri ID unik agar React key tidak bentrok jika ada soal duplikat (looping)
    const examQuestions = selectedQuestions.map((q, i) => ({
      ...q,
      id: `q-${Date.now()}-${i}`,
      realId: q.id
    }));

    setActiveExam({ ...pkg, questions: examQuestions });
    setExamState({
      currentQuestionIndex: 0,
      answers: {},
      timeLeft: pkg.time * 60, 
      isFinished: false,
      score: 0
    });
    setActiveTab('exam');
  };

  const answerQuestion = (qIndex, optionIndex) => {
    setExamState(prev => ({
      ...prev,
      answers: { ...prev.answers, [qIndex]: optionIndex }
    }));
  };

  const finishExam = () => {
    let correctCount = 0;
    activeExam.questions.forEach((q, idx) => {
      if (examState.answers[idx] === q.correct) {
        correctCount++;
      }
    });

    const rawScore = (correctCount / activeExam.questions.length) * 800 + 200;
    const finalScore = Math.round(rawScore);

    const newHistory = {
      id: Date.now(),
      date: new Date().toLocaleDateString('id-ID'),
      pkgTitle: activeExam.title,
      score: finalScore,
      correct: correctCount,
      total: activeExam.questions.length
    };

    setUserData(prev => ({
      ...prev,
      dailyProgress: prev.dailyProgress + activeExam.questions.length,
      history: [newHistory, ...prev.history]
    }));

    setExamState(prev => ({ ...prev, isFinished: true, score: finalScore }));
  };

  const exitExam = () => {
    if (window.confirm("Yakin ingin keluar? Progres tidak akan disimpan.")) {
      setActiveExam(null);
      setActiveTab('dashboard');
    }
  };

  const formatTime = (seconds) => {
    const m = Math.floor(seconds / 60);
    const s = seconds % 60;
    return `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
  };

  const renderDashboard = () => {
    const lastScore = userData.history.length > 0 ? userData.history[0].score : 0;
    const selectedUniv = UNIVERSITIES.find(u => u.id === userData.targetUniv);
    const selectedProdi = selectedUniv?.programs.find(p => p.name === userData.targetProdi);
    
    const chance = lastScore >= (selectedProdi?.passingGrade || 700) 
      ? { text: "Tinggi", color: "text-emerald-600", bg: "bg-emerald-100" }
      : lastScore >= ((selectedProdi?.passingGrade || 700) - 50)
      ? { text: "Sedang", color: "text-yellow-600", bg: "bg-yellow-100" }
      : { text: "Rendah", color: "text-rose-600", bg: "bg-rose-100" };

    return (
      <div className="space-y-6 animate-fade-in">
        <div className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
          <div>
            <h1 className="text-2xl font-bold text-slate-800">Halo, {userData.name}! 👋</h1>
            <p className="text-slate-500">Siap taklukkan UTBK hari ini?</p>
          </div>
          <div className="flex gap-3">
             <div className="bg-orange-100 px-4 py-2 rounded-lg flex items-center gap-2 text-orange-700 font-bold">
               <TrendingUp size={20} />
               <span>{userData.streak} Hari Streak</span>
             </div>
          </div>
        </div>

        <Card className="border-l-4 border-l-fuchsia-500 relative overflow-hidden">
          <div className="absolute top-0 right-0 p-4 opacity-10">
            <Sparkles size={100} className="text-fuchsia-500" />
          </div>
          <div className="relative z-10">
            <div className="flex justify-between items-start mb-4">
               <div>
                  <h3 className="text-lg font-bold text-slate-800 flex items-center gap-2">
                    <Sparkles className="text-fuchsia-500" size={20} />
                    Rencana Belajar Cerdas
                  </h3>
                  <p className="text-sm text-slate-500">Dibuat khusus oleh AI untuk target {selectedProdi?.name}</p>
               </div>
               {!userData.studyPlan && (
                 <Button onClick={generateStudyPlan} disabled={aiLoading} variant="ai" className="text-sm">
                   {aiLoading ? <><Loader2 className="animate-spin" size={16}/> Membuat...</> : 'Buat Rencana ✨'}
                 </Button>
               )}
               {userData.studyPlan && (
                 <Button onClick={generateStudyPlan} disabled={aiLoading} variant="secondary" className="text-xs">
                   <RefreshCw size={14}/> Refresh
                 </Button>
               )}
            </div>
            
            {userData.studyPlan ? (
              <div className="bg-fuchsia-50/50 rounded-lg p-4 border border-fuchsia-100">
                <MarkdownRenderer text={userData.studyPlan} />
              </div>
            ) : (
              <div className="bg-slate-50 rounded-lg p-6 text-center border border-dashed border-slate-300">
                <p className="text-slate-400 text-sm">Belum ada rencana belajar aktif. Klik tombol di atas untuk membuat jadwal otomatis!</p>
              </div>
            )}
          </div>
        </Card>

        <Card className="bg-gradient-to-r from-indigo-600 to-violet-600 text-white border-none">
          <div className="flex justify-between items-center mb-2">
            <h3 className="font-semibold flex items-center gap-2"><Target size={18}/> Target Harian</h3>
            <span className="text-indigo-100 text-sm">{userData.dailyProgress} / {userData.dailyTarget} Soal</span>
          </div>
          <ProgressBar value={userData.dailyProgress} max={userData.dailyTarget} color="bg-white/30" />
        </Card>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <Card>
            <div className="flex justify-between items-start mb-4">
              <div>
                <h3 className="text-lg font-semibold text-slate-800">Analisis Peluang</h3>
                <p className="text-sm text-slate-500">{selectedUniv?.name}</p>
                <p className="text-sm font-medium text-slate-700">{selectedProdi?.name}</p>
              </div>
              <GraduationCap className="text-indigo-500" size={24} />
            </div>
            <div className={`p-3 rounded-lg ${chance.bg} ${chance.color} font-bold text-center`}>
              Peluang: {chance.text}
            </div>
          </Card>

          <Card>
            <h3 className="text-lg font-semibold text-slate-800 mb-4">Riwayat Tryout</h3>
            <div className="space-y-3 max-h-48 overflow-y-auto pr-2">
              {userData.history.length === 0 ? (
                <p className="text-slate-400 text-sm text-center py-4">Belum ada riwayat tryout.</p>
              ) : (
                userData.history.map((h) => (
                  <div key={h.id} className="flex justify-between items-center p-3 bg-slate-50 rounded-lg border border-slate-100">
                    <div>
                      <p className="font-medium text-slate-700 text-sm">{h.pkgTitle}</p>
                      <p className="text-xs text-slate-400">{h.date}</p>
                    </div>
                    <div className="flex items-center gap-2">
                      <span className={`font-bold ${h.score >= 700 ? 'text-emerald-600' : 'text-slate-600'}`}>
                        {h.score}
                      </span>
                    </div>
                  </div>
                ))
              )}
            </div>
          </Card>
        </div>

        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
           {PACKAGES.slice(0,3).map((pkg) => (
             <button 
               key={pkg.id}
               onClick={() => startExam(pkg)}
               className="group flex flex-col p-4 bg-white border border-slate-200 rounded-xl hover:border-indigo-500 hover:shadow-md transition-all text-left"
             >
               <div className="flex justify-between items-start w-full mb-2">
                 <div className="p-2 bg-indigo-50 text-indigo-600 rounded-lg group-hover:bg-indigo-600 group-hover:text-white transition-colors">
                   <BookOpen size={20} />
                 </div>
                 <span className="text-xs font-medium bg-slate-100 px-2 py-1 rounded text-slate-600">{pkg.time} Menit</span>
               </div>
               <h4 className="font-bold text-slate-800 mb-1 line-clamp-1">{pkg.title}</h4>
               <p className="text-xs text-slate-500 mb-3">{pkg.description}</p>
               <div className="mt-auto flex items-center text-indigo-600 text-sm font-medium">
                 Mulai Sekarang <ChevronRight size={16} />
               </div>
             </button>
           ))}
        </div>
      </div>
    );
  };

  const renderExam = () => {
    const currentQ = activeExam.questions[examState.currentQuestionIndex];
    const isAnswered = examState.answers[examState.currentQuestionIndex] !== undefined;

    if (examState.isFinished) {
      return (
        <div className="max-w-2xl mx-auto space-y-6 animate-fade-in">
          <Card className="text-center py-10">
            <Award size={40} className="mx-auto text-indigo-600 mb-6" />
            <h2 className="text-3xl font-bold text-slate-800 mb-2">Tryout Selesai!</h2>
            <div className="text-5xl font-black text-indigo-600 mb-2">{examState.score}</div>
            <div className="flex gap-4 justify-center mt-8">
              <Button onClick={() => { setActiveExam(null); setActiveTab('dashboard'); }} variant="secondary">
                <Home size={18} /> Kembali
              </Button>
              <Button onClick={() => setActiveTab('review')}>
                <BookOpen size={18} /> Pembahasan
              </Button>
            </div>
          </Card>
        </div>
      );
    }

    return (
      <div className="h-[calc(100vh-100px)] flex flex-col md:flex-row gap-6 animate-fade-in">
        <div className="flex-1 flex flex-col">
          <Card className="flex-1 flex flex-col overflow-hidden">
            <div className="flex justify-between items-center mb-4 pb-4 border-b border-slate-100">
               <span className="text-sm font-bold text-indigo-600 bg-indigo-50 px-3 py-1 rounded-full">
                 {currentQ.type}
               </span>
               <span className="text-slate-400 text-sm">Soal {examState.currentQuestionIndex + 1} dari {activeExam.questions.length}</span>
            </div>

            <div className="flex-1 overflow-y-auto pr-2">
              <p className="text-lg text-slate-800 leading-relaxed mb-6 font-medium">
                {currentQ.question}
              </p>
              
              <div className="space-y-3">
                {currentQ.options.map((opt, idx) => (
                  <button
                    key={idx}
                    onClick={() => answerQuestion(examState.currentQuestionIndex, idx)}
                    className={`w-full text-left p-4 rounded-xl border transition-all flex items-start gap-3
                      ${examState.answers[examState.currentQuestionIndex] === idx 
                        ? 'border-indigo-500 bg-indigo-50 text-indigo-700 ring-1 ring-indigo-500' 
                        : 'border-slate-200 hover:border-indigo-300 hover:bg-slate-50 text-slate-700'
                      }`}
                  >
                    <div className={`w-6 h-6 rounded-full flex items-center justify-center text-xs font-bold border flex-shrink-0
                       ${examState.answers[examState.currentQuestionIndex] === idx 
                         ? 'bg-indigo-600 border-indigo-600 text-white' 
                         : 'bg-white border-slate-300 text-slate-500'
                       }`}>
                       {String.fromCharCode(65 + idx)}
                    </div>
                    <span>{opt}</span>
                  </button>
                ))}
              </div>
            </div>

            <div className="pt-6 mt-4 border-t border-slate-100 flex justify-between">
              <Button 
                variant="secondary" 
                onClick={() => setExamState(prev => ({...prev, currentQuestionIndex: Math.max(0, prev.currentQuestionIndex - 1)}))}
                disabled={examState.currentQuestionIndex === 0}
              >
                Sebelumnya
              </Button>
              <Button 
                 onClick={() => setExamState(prev => ({...prev, currentQuestionIndex: Math.min(activeExam.questions.length - 1, prev.currentQuestionIndex + 1)}))}
                 disabled={examState.currentQuestionIndex === activeExam.questions.length - 1}
              >
                Selanjutnya
              </Button>
            </div>
          </Card>
        </div>

        <div className="w-full md:w-80 flex flex-col gap-4">
          <Card className="bg-slate-800 text-white border-none">
            <div className="text-center">
              <p className="text-slate-400 text-xs mb-1 uppercase tracking-widest">Sisa Waktu</p>
              <div className="text-4xl font-mono font-bold tracking-wider text-white">
                {formatTime(examState.timeLeft)}
              </div>
            </div>
          </Card>

          <Card className="flex-1 overflow-hidden flex flex-col">
            <h3 className="font-semibold text-slate-800 mb-4">Navigasi Soal</h3>
            <div className="grid grid-cols-5 gap-2 overflow-y-auto content-start">
              {activeExam.questions.map((_, idx) => (
                <button
                  key={idx}
                  onClick={() => setExamState(prev => ({...prev, currentQuestionIndex: idx}))}
                  className={`aspect-square rounded flex items-center justify-center text-sm font-bold transition-all
                    ${examState.currentQuestionIndex === idx 
                      ? 'ring-2 ring-indigo-500 ring-offset-2' 
                      : ''}
                    ${examState.answers[idx] !== undefined
                      ? 'bg-indigo-600 text-white'
                      : 'bg-slate-100 text-slate-500 hover:bg-slate-200'
                    }
                  `}
                >
                  {idx + 1}
                </button>
              ))}
            </div>
            <div className="mt-6 pt-4 border-t border-slate-100">
              <Button variant="success" className="w-full" onClick={finishExam}>
                Kumpulkan Jawaban
              </Button>
              <button onClick={exitExam} className="mt-3 w-full text-center text-slate-400 text-xs hover:text-rose-500 transition-colors">
                Batalkan Ujian
              </button>
            </div>
          </Card>
        </div>
      </div>
    );
  };

  const renderReview = () => {
    return (
      <div className="max-w-4xl mx-auto space-y-6 animate-fade-in pb-20">
        <div className="flex items-center justify-between">
           <h2 className="text-2xl font-bold text-slate-800">Pembahasan & Kunci Jawaban</h2>
           <Button variant="secondary" onClick={() => { setActiveExam(null); setActiveTab('dashboard'); }}>
             Tutup Pembahasan
           </Button>
        </div>

        {activeExam.questions.map((q, idx) => {
           const userAnswer = examState.answers[idx];
           const isCorrect = userAnswer === q.correct;
           const isAiLoadingThis = aiExplanation?.id === q.id && aiExplanation?.loading;
           const aiData = aiExplanation?.id === q.id ? aiExplanation.text : null;
           
           return (
             <Card key={idx} className={`border-l-4 ${isCorrect ? 'border-l-emerald-500' : 'border-l-rose-500'}`}>
                <div className="flex gap-2 items-center mb-2">
                  <span className="font-bold text-slate-700">Soal {idx + 1}</span>
                  {isCorrect ? (
                    <span className="bg-emerald-100 text-emerald-700 text-xs px-2 py-0.5 rounded flex items-center gap-1"><CheckCircle size={12}/> Benar</span>
                  ) : (
                    <span className="bg-rose-100 text-rose-700 text-xs px-2 py-0.5 rounded flex items-center gap-1"><XCircle size={12}/> Salah</span>
                  )}
                </div>
                <p className="text-slate-800 mb-4">{q.question}</p>
                
                <div className="bg-slate-50 p-4 rounded-lg mb-4">
                  <p className="text-xs text-slate-400 font-bold uppercase mb-2">Jawaban Kamu</p>
                  <p className={`${isCorrect ? 'text-emerald-700' : 'text-rose-600'}`}>
                    {userAnswer !== undefined ? q.options[userAnswer] : 'Tidak dijawab'}
                  </p>
                </div>

                <div className="bg-indigo-50 p-4 rounded-lg mb-4">
                   <p className="text-xs text-indigo-400 font-bold uppercase mb-2">Kunci Jawaban & Pembahasan</p>
                   <p className="font-semibold text-indigo-900 mb-2">{q.options[q.correct]}</p>
                   <p className="text-sm text-indigo-800 leading-relaxed">{q.explanation}</p>
                </div>

                <div>
                   {!aiData && !isAiLoadingThis && (
                     <Button variant="ai" onClick={() => getAiExplanation(q)} className="w-full sm:w-auto text-sm">
                       <Sparkles size={16} /> Tanya AI (Penjelasan Detail) ✨
                     </Button>
                   )}
                   
                   {isAiLoadingThis && (
                     <div className="flex items-center gap-2 text-fuchsia-600 text-sm animate-pulse p-2">
                       <Loader2 size={16} className="animate-spin" /> Sedang menghubungi Tutor AI...
                     </div>
                   )}

                   {aiData && (
                     <div className="mt-4 bg-gradient-to-br from-fuchsia-50 to-violet-50 border border-fuchsia-100 rounded-lg p-4 animate-fade-in">
                       <div className="flex items-center gap-2 text-fuchsia-700 font-bold mb-2 text-sm">
                         <Sparkles size={16} /> Penjelasan Tutor AI
                       </div>
                       <MarkdownRenderer text={aiData} />
                     </div>
                   )}
                </div>
             </Card>
           )
        })}
      </div>
    )
  }

  const renderPaketList = () => (
    <div className="space-y-6 animate-fade-in">
      <div className="flex items-center justify-between">
        <h2 className="text-2xl font-bold text-slate-800">Pilih Paket Tryout</h2>
        <div className="flex gap-2">
          {['Semua', 'Saintek', 'Soshum'].map(filter => (
            <button key={filter} className="px-3 py-1 rounded-full text-sm bg-white border hover:bg-slate-50">{filter}</button>
          ))}
        </div>
      </div>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {PACKAGES.map(pkg => (
          <Card key={pkg.id} className="hover:shadow-md transition-shadow cursor-pointer border-l-4 border-l-indigo-500">
             <div className="flex justify-between items-start">
                <div>
                   <h3 className="font-bold text-lg text-slate-800">{pkg.title}</h3>
                   <div className="flex gap-3 mt-2 text-sm text-slate-500">
                      <span className="flex items-center gap-1"><Clock size={14}/> {pkg.time} Menit</span>
                      <span className="flex items-center gap-1"><BookOpen size={14}/> {pkg.count} Soal</span>
                   </div>
                   <p className="mt-3 text-slate-600 text-sm">{pkg.description}</p>
                </div>
                <button onClick={() => startExam(pkg)} className="bg-indigo-600 text-white p-3 rounded-full shadow-lg shadow-indigo-200 hover:scale-105 transition-transform">
                  <Play size={20} fill="currentColor" />
                </button>
             </div>
          </Card>
        ))}
      </div>
    </div>
  );

  const renderSettings = () => (
    <div className="max-w-xl mx-auto space-y-6 animate-fade-in">
      <h2 className="text-2xl font-bold text-slate-800">Profil & Target</h2>
      <Card>
         <div className="space-y-4">
           <div>
             <label className="block text-sm font-medium text-slate-700 mb-1">Nama</label>
             <input type="text" className="w-full p-2 border border-slate-300 rounded-lg" value={userData.name} onChange={(e) => setUserData({...userData, name: e.target.value})} />
           </div>
           <div>
             <label className="block text-sm font-medium text-slate-700 mb-1">Target Universitas</label>
             <select className="w-full p-2 border border-slate-300 rounded-lg" value={userData.targetUniv} onChange={(e) => setUserData({...userData, targetUniv: e.target.value, targetProdi: UNIVERSITIES.find(u => u.id === e.target.value).programs[0].name})}>
               {UNIVERSITIES.map(u => (<option key={u.id} value={u.id}>{u.name}</option>))}
             </select>
           </div>
           <div>
             <label className="block text-sm font-medium text-slate-700 mb-1">Target Program Studi</label>
             <select className="w-full p-2 border border-slate-300 rounded-lg" value={userData.targetProdi} onChange={(e) => setUserData({...userData, targetProdi: e.target.value})}>
               {UNIVERSITIES.find(u => u.id === userData.targetUniv)?.programs.map(p => (<option key={p.name} value={p.name}>{p.name} (Grade: {p.passingGrade})</option>))}
             </select>
           </div>
           <Button className="w-full justify-center mt-4"><Save size={18} /> Simpan Perubahan</Button>
         </div>
      </Card>
      <Card className="bg-rose-50 border-rose-100">
        <h3 className="font-bold text-rose-700 mb-2 flex items-center gap-2"><AlertCircle size={18}/> Zona Bahaya</h3>
        <Button variant="danger" onClick={() => { if(confirm("Hapus semua data?")) { localStorage.removeItem('utbkProData'); window.location.reload(); } }}>Reset Semua Data</Button>
      </Card>
    </div>
  );

  return (
    <div className="min-h-screen bg-slate-50 font-sans text-slate-900 flex">
      <aside className={`fixed md:relative z-20 h-full w-20 md:w-64 bg-slate-900 text-white flex flex-col transition-all ${activeTab === 'exam' ? 'hidden md:flex' : ''}`}>
        <div className="p-4 md:p-6 flex items-center gap-3 font-bold text-xl md:text-2xl text-indigo-400">
           <div className="w-8 h-8 bg-indigo-500 rounded-lg flex items-center justify-center text-white"><BookOpen size={20} /></div>
           <span className="hidden md:block">UTBK<span className="text-white">Pro</span></span>
        </div>
        <nav className="flex-1 px-2 md:px-4 py-6 space-y-2">
           <button onClick={() => activeTab !== 'exam' && setActiveTab('dashboard')} className={`w-full flex items-center gap-3 p-3 rounded-xl transition-colors ${activeTab === 'dashboard' ? 'bg-indigo-600 text-white shadow-lg shadow-indigo-900' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}><Home size={20} /> <span className="hidden md:block">Dashboard</span></button>
           <button onClick={() => activeTab !== 'exam' && setActiveTab('packages')} className={`w-full flex items-center gap-3 p-3 rounded-xl transition-colors ${activeTab === 'packages' ? 'bg-indigo-600 text-white shadow-lg shadow-indigo-900' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}><Play size={20} /> <span className="hidden md:block">Tryout & Drill</span></button>
           <button onClick={() => activeTab !== 'exam' && setActiveTab('settings')} className={`w-full flex items-center gap-3 p-3 rounded-xl transition-colors ${activeTab === 'settings' ? 'bg-indigo-600 text-white shadow-lg shadow-indigo-900' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}><Target size={20} /> <span className="hidden md:block">Target Kampus</span></button>
        </nav>
        <div className="p-4 text-xs text-slate-500 text-center hidden md:block">v1.2.0 Fixed</div>
      </aside>
      <main className="flex-1 h-screen overflow-y-auto overflow-x-hidden p-4 md:p-8">
        {activeTab === 'dashboard' && renderDashboard()}
        {activeTab === 'packages' && renderPaketList()}
        {activeTab === 'exam' && renderExam()}
        {activeTab === 'review' && renderReview()}
        {activeTab === 'settings' && renderSettings()}
      </main>
      <style>{`@keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } } .animate-fade-in { animation: fade-in 0.3s ease-out forwards; }`}</style>
    </div>
  );
}
