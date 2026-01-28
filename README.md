
import React, { useState } from 'react';
import { UserProfile, Tone, ThaiIntroduction, AppState } from './types';
import { generateThaiIntro, playThaiAudio } from './services/geminiService';

const App: React.FC = () => {
  const [state, setState] = useState<AppState>({
    profile: {
      name: 'HuangZhiRong',
      nickname: 'ZhiRong',
      occupation: 'นักศึกษาสาขาภาษาไทย',
      hobbies: 'ฟังเพลงและดูภาพยนตร์',
      residence: '',
      origin: '',
      goal: 'ยินดีที่ได้รู้จักทุกคน'
    },
    introduction: null,
    isLoading: false,
    isAudioLoading: false,
    error: null,
    selectedTone: Tone.GENZ
  });

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => {
    const { name, value } = e.target;
    setState(prev => ({
      ...prev,
      profile: { ...prev.profile, [name]: value }
    }));
  };

  const handleGenerate = async (e: React.FormEvent) => {
    e.preventDefault();
    setState(prev => ({ ...prev, isLoading: true, error: null }));
    try {
      const intro = await generateThaiIntro(state.profile, state.selectedTone);
      setState(prev => ({ ...prev, introduction: intro, isLoading: false }));
    } catch (err) {
      setState(prev => ({ ...prev, isLoading: false, error: 'เกิดข้อผิดพลาดในการสร้างข้อมูล กรุณาลองใหม่อีกครั้ง' }));
    }
  };

  const handlePlayAudio = async () => {
    if (!state.introduction) return;
    setState(prev => ({ ...prev, isAudioLoading: true }));
    try {
      await playThaiAudio(state.introduction.thaiText);
    } catch (err) {
      console.error(err);
    } finally {
      setState(prev => ({ ...prev, isAudioLoading: false }));
    }
  };

  return (
    <div className="min-h-screen pb-20">
      {/* Dynamic Header */}
      <header className="relative pt-16 pb-20 px-4 overflow-hidden text-center">
        <div className="absolute top-0 left-1/2 -translate-x-1/2 w-full h-full -z-10 opacity-20 blur-3xl scale-150 pointer-events-none">
          <div className="absolute top-0 right-0 w-96 h-96 bg-pink-500 rounded-full"></div>
          <div className="absolute bottom-0 left-0 w-96 h-96 bg-purple-600 rounded-full"></div>
        </div>
        
        <div className="inline-block px-4 py-1 rounded-full bg-white/50 backdrop-blur-sm border border-white/30 text-xs font-bold text-purple-600 mb-4 tracking-widest uppercase">
          Vibe Check ✨
        </div>
        <h1 className="text-5xl md:text-7xl font-bold mb-6 tracking-tighter">
          <span className="text-gradient">แนะนำตัวแบบสับ</span>
        </h1>
        <p className="max-w-xl mx-auto text-gray-600 text-lg md:text-xl font-medium leading-relaxed">
          สร้างบทแนะนำตัวสุดคูลแบบวัยรุ่นไทย พร้อมเช็คคำอ่านและเสียงพูดอัตโนมัติ
        </p>
      </header>

      <main className="max-w-6xl mx-auto px-4 grid grid-cols-1 lg:grid-cols-2 gap-10">
        {/* Left: Input Form */}
        <section className="glass p-8 md:p-10 rounded-[2.5rem] shadow-2xl">
          <div className="flex items-center gap-3 mb-8">
            <div className="w-12 h-12 rounded-2xl gen-z-gradient flex items-center justify-center shadow-lg">
              <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
              </svg>
            </div>
            <h2 className="text-2xl font-bold text-gray-800">ข้อมูลส่วนตัว</h2>
          </div>

          <form onSubmit={handleGenerate} className="space-y-6">
            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div className="space-y-2">
                <label className="text-sm font-bold text-gray-500 ml-1">ชื่อ-นามสกุล</label>
                <input
                  required
                  name="name"
                  value={state.profile.name}
                  onChange={handleInputChange}
                  placeholder="เช่น สมชาย ใจดี"
                  className="w-full px-5 py-4 rounded-2xl bg-white border-2 border-transparent focus:border-purple-400 shadow-sm transition-all outline-none"
                />
              </div>
              <div className="space-y-2">
                <label className="text-sm font-bold text-gray-500 ml-1">ชื่อเล่น</label>
                <input
                  name="nickname"
                  value={state.profile.nickname}
                  onChange={handleInputChange}
                  placeholder="เช่น ซันนี่"
                  className="w-full px-5 py-4 rounded-2xl bg-white border-2 border-transparent focus:border-purple-400 shadow-sm transition-all outline-none"
                />
              </div>
            </div>

            <div className="space-y-2">
              <label className="text-sm font-bold text-gray-500 ml-1">สไตล์ภาษา</label>
              <div className="grid grid-cols-3 gap-3">
                {[
                  { id: Tone.GENZ, label: 'วัยรุ่น' },
                  { id: Tone.CASUAL, label: 'กันเอง' },
                  { id: Tone.FORMAL, label: 'สุภาพ' }
                ].map((t) => (
                  <button
                    key={t.id}
                    type="button"
                    onClick={() => setState(prev => ({ ...prev, selectedTone: t.id }))}
                    className={`py-3 rounded-2xl text-sm font-bold transition-all border-2 ${
                      state.selectedTone === t.id 
                      ? 'bg-purple-600 border-purple-600 text-white shadow-lg' 
                      : 'bg-white border-gray-100 text-gray-500 hover:border-purple-200'
                    }`}
                  >
                    {t.label}
                  </button>
                ))}
              </div>
            </div>

            <div className="space-y-2">
              <label className="text-sm font-bold text-gray-500 ml-1">การศึกษา / อาชีพ</label>
              <input
                name="occupation"
                value={state.profile.occupation}
                onChange={handleInputChange}
                placeholder="คุณทำอะไรอยู่?"
                className="w-full px-5 py-4 rounded-2xl bg-white border-2 border-transparent focus:border-purple-400 shadow-sm transition-all outline-none"
              />
            </div>

            <div className="space-y-2">
              <label className="text-sm font-bold text-gray-500 ml-1">งานอดิเรก</label>
              <textarea
                name="hobbies"
                value={state.profile.hobbies}
                onChange={handleInputChange}
                rows={2}
                placeholder="ทำอาหาร, เขียนโค้ด, ต่อยมวย..."
                className="w-full px-5 py-4 rounded-2xl bg-white border-2 border-transparent focus:border-purple-400 shadow-sm transition-all outline-none resize-none"
              />
            </div>

            <div className="space-y-2">
              <label className="text-sm font-bold text-gray-500 ml-1">เป้าหมาย / ความรู้สึก</label>
              <input
                name="goal"
                value={state.profile.goal}
                onChange={handleInputChange}
                placeholder="หาเพื่อนใหม่ / สมัครงาน..."
                className="w-full px-5 py-4 rounded-2xl bg-white border-2 border-transparent focus:border-purple-400 shadow-sm transition-all outline-none"
              />
            </div>

            <button
              type="submit"
              disabled={state.isLoading}
              className={`w-full py-5 rounded-3xl font-bold text-white shadow-xl transition-all transform flex items-center justify-center gap-3 ${
                state.isLoading ? 'bg-gray-400 cursor-not-allowed' : 'gen-z-gradient hover:scale-[1.02] active:scale-[0.98] hover:shadow-purple-500/30'
              }`}
            >
              {state.isLoading ? (
                <div className="flex items-center gap-2">
                  <div className="w-5 h-5 border-2 border-white/30 border-t-white rounded-full animate-spin"></div>
                  <span>กำลังประมวลผล...</span>
                </div>
              ) : (
                <>
                  <span className="text-lg">เริ่มสร้างบทแนะนำตัว</span>
                  <span className="text-2xl">🔥</span>
                </>
              )}
            </button>
          </form>
        </section>

        {/* Right: Output Result */}
        <section className="space-y-8">
          {!state.introduction && !state.isLoading && (
            <div className="h-full min-h-[500px] flex flex-col items-center justify-center glass rounded-[2.5rem] p-12 text-center border-dashed border-2 border-purple-200">
              <div className="w-24 h-24 bg-white rounded-full flex items-center justify-center shadow-inner mb-6 animate-bounce">
                <span className="text-5xl">👋</span>
              </div>
              <h3 className="text-2xl font-bold text-gray-800 mb-4">พร้อมจะแนะนำตัวหรือยัง?</h3>
              <p className="text-gray-500 max-w-xs leading-relaxed">
                ข้อมูลของคุณ <strong>{state.profile.name}</strong> ถูกกรอกไว้แล้ว กดปุ่มสร้างเพื่อดูผลลัพธ์สุดว้าวได้เลย!
              </p>
            </div>
          )}

          {state.introduction && (
            <div className="animate-in fade-in zoom-in-95 duration-700">
              <div className="glass rounded-[2.5rem] shadow-2xl overflow-hidden border-2 border-white/50">
                <div className="bg-white/40 px-8 py-6 flex justify-between items-center border-b border-white/30">
                  <span className="px-3 py-1 rounded-full bg-purple-100 text-purple-600 text-xs font-bold uppercase tracking-widest">
                    ผลลัพธ์
                  </span>
                  <button
                    onClick={handlePlayAudio}
                    disabled={state.isAudioLoading}
                    className="flex items-center gap-2 px-6 py-3 gen-z-gradient text-white font-bold rounded-2xl hover:opacity-90 transition-all shadow-lg active:scale-95 disabled:opacity-50"
                  >
                    {state.isAudioLoading ? 'กำลังโหลด...' : (
                      <>
                        <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z" />
                        </svg>
                        ฟังเสียง
                      </>
                    )}
                  </button>
                </div>
                
                <div className="p-8 md:p-10 space-y-10">
                  {/* Thai Text */}
                  <div>
                    <h4 className="text-[10px] font-bold text-gray-400 uppercase mb-3 tracking-[0.2em] flex items-center gap-2">
                      <span className="w-1 h-1 bg-purple-500 rounded-full"></span>
                      ภาษาไทย
                    </h4>
                    <p className="text-3xl md:text-4xl font-bold text-gray-900 leading-[1.4]">
                      {state.introduction.thaiText}
                    </p>
                  </div>

                  {/* Phonetic */}
                  <div className="p-6 bg-gradient-to-br from-purple-50 to-pink-50 rounded-3xl border border-white/50 shadow-inner">
                    <h4 className="text-[10px] font-bold text-purple-500 uppercase mb-2 tracking-[0.2em]">คำอ่าน (โฟเนติก)</h4>
                    <p className="text-xl italic text-gray-700 leading-relaxed font-medium">
                      {state.introduction.phonetic}
                    </p>
                  </div>

                  {/* English Translation */}
                  <div>
                    <h4 className="text-[10px] font-bold text-gray-400 uppercase mb-3 tracking-[0.2em] flex items-center gap-2">
                      <span className="w-1 h-1 bg-pink-500 rounded-full"></span>
                      คำแปลอังกฤษ
                    </h4>
                    <p className="text-lg text-gray-600 leading-relaxed font-medium">
                      {state.introduction.englishTranslation}
                    </p>
                  </div>

                  {/* Cultural Tips */}
                  <div className="pt-8 border-t border-gray-100">
                    <h4 className="text-[10px] font-bold text-gray-400 uppercase mb-4 tracking-[0.2em]">สาระน่ารู้</h4>
                    <div className="grid grid-cols-1 gap-3">
                      {state.introduction.culturalTips.map((tip, idx) => (
                        <div key={idx} className="flex items-center gap-4 p-4 rounded-2xl bg-white/30 border border-white/50 text-sm text-gray-700">
                          <span className="flex-shrink-0 w-8 h-8 rounded-full bg-white flex items-center justify-center font-bold text-purple-600 shadow-sm">
                            {idx + 1}
                          </span>
                          {tip}
                        </div>
                      ))}
                    </div>
                  </div>
                </div>
              </div>
            </div>
          )}

          {state.error && (
            <div className="p-6 bg-red-50 border border-red-100 text-red-600 rounded-3xl flex items-center gap-4 shadow-xl shadow-red-500/5">
              <span className="text-2xl">⚠️</span>
              <p className="font-bold">{state.error}</p>
            </div>
          )}
        </section>
      </main>

      {/* Footer */}
      <footer className="mt-20 py-10 border-t border-gray-100 text-center">
        <p className="text-gray-400 font-medium tracking-tight">
          สร้างสรรค์ด้วยใจโดย <span className="font-bold text-purple-500">วัยรุ่นไท</span> © {new Date().getFullYear()}
        </p>
      </footer>
    </div>
  );
};

export default App;
