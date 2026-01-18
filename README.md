<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DR. CHU UROLOGY | 陰莖美型專家</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <!-- Babel for JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <style>
        .scrollbar-hide::-webkit-scrollbar {
            display: none;
        }
        .scrollbar-hide {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        body {
            background-color: #f8fafc;
            -webkit-tap-highlight-color: transparent;
        }
        .active-date-scroll {
            scroll-behavior: smooth;
        }
        button {
            user-select: none;
        }
        .fade-in {
            animation: fadeIn 0.3s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;
        
        // --- 內嵌圖示元件 (確保單檔穩定) ---
        const Icon = ({ name, size = 24, className = "" }) => {
            const icons = {
                Activity: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M22 12h-4l-3 9L9 3l-3 9H2"/></svg>,
                Clock: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>,
                ChevronRight: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m9 18 6-6-6-6"/></svg>,
                ChevronLeft: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m15 18-6-6 6-6"/></svg>,
                CheckCircle: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/></svg>,
                MessageCircle: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m3 21 1.9-5.7a8.5 8.5 0 1 1 3.8 3.8z"/></svg>,
                Calendar: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><rect width="18" height="18" x="3" y="4" rx="2" ry="2"/><line x1="16" x2="16" y1="2" y2="6"/><line x1="8" x2="8" y1="2" y2="6"/><line x1="3" x2="21" y1="10" y2="10"/></svg>,
                Check: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><polyline points="20 6 9 17 4 12"/></svg>,
                Globe: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><circle cx="12" cy="12" r="10"/><line x1="2" x2="22" y1="12" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/></svg>,
            };
            return icons[name] || null;
        };

        const Clock = (props) => <Icon name="Clock" {...props} />;
        const ChevronRight = (props) => <Icon name="ChevronRight" {...props} />;
        const ChevronLeft = (props) => <Icon name="ChevronLeft" {...props} />;
        const CheckCircle = (props) => <Icon name="CheckCircle" {...props} />;
        const MessageCircle = (props) => <Icon name="MessageCircle" {...props} />;
        const Calendar = (props) => <Icon name="Calendar" {...props} />;
        const Check = (props) => <Icon name="Check" {...props} />;

        // --- 每週班表配置 ---
        const WEEKLY_SESSIONS = {
          1: ['afternoon', 'evening'], // 週一 (下午、晚上)
          2: [],                       // 週二 (全日無門診)
          3: ['afternoon', 'evening'], // 週三 (下午、晚上)
          4: ['morning', 'afternoon'], // 週四 (上午、下午)
          5: ['morning'],              // 週五 (上午)
          6: ['afternoon'],            // 週六 (下午)
          0: [],                       // 週日 (休診)
        };

        const SESSION_TIMES = {
          morning:   { start: '09:00', end: '12:00', interval: 30 },
          afternoon: { start: '14:00', end: '17:00', interval: 30 },
          evening:   { start: '18:00', end: '20:30', interval: 30 },
        };
        
        // 諮詢項目
        const SERVICE_OPTIONS = [
          { id: 'enlarge', zh: '陰莖增粗', en: 'Penis Enlargement' },
          { id: 'glans', zh: '龜頭撐大', en: 'Glans Enhancement' },
          { id: 'hardness', zh: '增強硬度', en: 'Hardness Enhancement' },
          { id: 'circumcision', zh: '包皮手術', en: 'Circumcision' },
          { id: 'premature', zh: '早洩治療', en: 'Premature Ejaculation' },
          { id: 'ligation', zh: '結紮手術', en: 'Vasectomy' },
        ];

        const LINE_ID = "@565wgjes";
        const LINE_URL = "https://line.me/R/ti/p/@565wgjes"; 

        const TRANSLATIONS = {
          zh: {
            navTitle: "DR. CHU UROLOGY",
            subTitle: "陰莖美型專家",
            consult: "線上諮詢",
            drName: "朱信誠 醫師",
            drTitle: "Dr. Chu Hsin-Cheng",
            specialty: "陰莖美型專家",
            hoursLabel: "門診時間：週一至週六 (週二、日休)",
            description: "男性自信是外在與內在的融合，秉持實在、精緻、溫柔、美型的精神，藉由私密手術以及性功能促進，成功讓男性重獲自信",
            note: "※ 灰色時段代表已額滿，建議您提早預約。",
            weekTitle: "預約門診",
            closedToday: "本日休診",
            selectTime: "選擇看診時段",
            full: "額滿",
            available: "預約",
            noSlots: "當日無門診時段",
            selectedLabel: "您的選擇",
            selectPrompt: "請選擇上方日期與時段",
            bookBtn: "確認預約",
            modalTitle: "填寫預約資訊",
            labelName: "預約大名",
            placeholderName: "請輸入您的真實姓名",
            labelPhone: "聯絡電話",
            labelService: "諮詢項目 (可複選)",
            confirmBtn: "送出預約",
            cancelBtn: "返回",
            successTitle: "預約成功",
            successDesc: "我們已收到您的預約資訊。\n請務必加入官方 LINE 以確認看診序號。",
            lineIdLabel: "官方 LINE ID",
            addLineBtn: "加入 LINE 好友",
            closeBtn: "回到首頁",
            weekDays: ['日','一','二','三','四','五','六'],
            dayPrefix: "週",
            yearSuffix: "年",
            monthSuffix: "月"
          },
          en: {
            navTitle: "DR. CHU UROLOGY",
            subTitle: "Penile Aesthetics",
            consult: "Inquiry",
            drName: "Dr. Chu Hsin-Cheng",
            drTitle: "Urologist",
            specialty: "Specialist",
            hoursLabel: "Hours: Mon-Sat (Tue/Sun Closed)",
            description: "Male confidence is a fusion of exterior and interior. Specialized in intimate surgery and sexual function promotion.",
            note: "※ Gray slots indicate full.",
            weekTitle: "Appointment",
            closedToday: "Closed Today",
            selectTime: "Select Time",
            full: "FULL",
            available: "BOOK",
            noSlots: "No slots today",
            selectedLabel: "Selected",
            selectPrompt: "Please select time",
            bookBtn: "Confirm",
            modalTitle: "Booking Info",
            labelName: "Name",
            placeholderName: "Full Name",
            labelPhone: "Phone",
            labelService: "Service (Multiple)",
            confirmBtn: "Submit",
            cancelBtn: "Back",
            successTitle: "Success",
            successDesc: "Received. Please add LINE for confirmation.",
            lineIdLabel: "Official LINE ID",
            addLineBtn: "Add LINE",
            closeBtn: "Home",
            weekDays: ['Sun','Mon','Tue','Wed','Thu','Fri','Sat'],
            dayPrefix: "",
            yearSuffix: "",
            monthSuffix: ""
          }
        };

        const generateTimeSlots = (sessionType) => {
          const { start, end, interval } = SESSION_TIMES[sessionType];
          const slots = [];
          let currentTime = new Date(`2000-01-01T${start}`);
          const endTime = new Date(`2000-01-01T${end}`);

          while (currentTime <= endTime) {
            const timeString = currentTime.toTimeString().slice(0, 5);
            // 隨機產生 20% 的額滿率 (演示用)
            const isFull = Math.random() < 0.2; 
            slots.push({ time: timeString, status: isFull ? 'full' : 'available' });
            currentTime.setMinutes(currentTime.getMinutes() + interval);
          }
          return slots;
        };

        function MedicalBookingApp() {
          const [lang, setLang] = useState('zh'); 
          const t = TRANSLATIONS[lang];
          const dateScrollRef = useRef(null);

          const [viewDate, setViewDate] = useState(new Date()); 
          const [selectedDateIndex, setSelectedDateIndex] = useState(null);
          const [currentDaySlots, setCurrentDaySlots] = useState([]);
          const [selectedTime, setSelectedTime] = useState(null);
          const [selectedServices, setSelectedServices] = useState([]);
          const [showModal, setShowModal] = useState(false);
          const [step, setStep] = useState(1);

          const toggleLang = () => setLang(prev => prev === 'zh' ? 'en' : 'zh');

          const generateMonthDates = (baseDate) => {
            const year = baseDate.getFullYear();
            const month = baseDate.getMonth();
            const dates = [];
            const today = new Date();
            today.setHours(0, 0, 0, 0); 
            const daysInMonth = new Date(year, month + 1, 0).getDate();

            for (let day = 1; day <= daysInMonth; day++) {
              const d = new Date(year, month, day);
              dates.push({
                fullDate: d,
                label: `${month + 1}/${day}`,
                dayName: t.weekDays[d.getDay()], 
                dayIndex: d.getDay(),
                isToday: d.getTime() === today.getTime(),
                isPast: d < today
              });
            }
            return dates;
          };

          const availableDates = generateMonthDates(viewDate);
          const currentDateObj = selectedDateIndex !== null ? availableDates[selectedDateIndex] : null;

          // 初始化：定位至今日
          useEffect(() => {
             const todayIndex = availableDates.findIndex(d => d.isToday);
             if (todayIndex !== -1) {
                 setSelectedDateIndex(todayIndex);
                 setTimeout(() => scrollToIndex(todayIndex), 150);
             } else {
                 const firstAvailable = availableDates.findIndex(d => !d.isPast);
                 const target = firstAvailable !== -1 ? firstAvailable : 0;
                 setSelectedDateIndex(target);
                 setTimeout(() => scrollToIndex(target), 150);
             }
          }, [viewDate, lang]); 

          const scrollToIndex = (index) => {
             if (dateScrollRef.current) {
                const element = dateScrollRef.current.children[index];
                if (element) {
                    const containerWidth = dateScrollRef.current.offsetWidth;
                    const elementOffset = element.offsetLeft;
                    const elementWidth = element.offsetWidth;
                    dateScrollRef.current.scrollTo({
                        left: elementOffset - (containerWidth / 2) + (elementWidth / 2),
                        behavior: 'smooth'
                    });
                }
             }
          };

          const scrollDates = (direction) => {
            if (dateScrollRef.current) {
                const scrollAmount = dateScrollRef.current.offsetWidth * 0.7;
                dateScrollRef.current.scrollBy({ left: direction === 'right' ? scrollAmount : -scrollAmount, behavior: 'smooth' });
            }
          };

          useEffect(() => {
              if (currentDateObj) {
                  const sessions = WEEKLY_SESSIONS[currentDateObj.dayIndex] || [];
                  const slots = sessions.reduce((acc, sess) => [...acc, ...generateTimeSlots(sess)], []);
                  setCurrentDaySlots(slots);
              }
          }, [selectedDateIndex, viewDate]);

          const handleTimeSelect = (slot) => {
            if (slot.status === 'full') return;
            setSelectedTime(slot.time);
          };
          
          const handleServiceToggle = (id) => {
            setSelectedServices(prev => prev.includes(id) ? prev.filter(x => x !== id) : [...prev, id]);
          };

          const nextMonth = () => { setViewDate(prev => { const d = new Date(prev); d.setMonth(prev.getMonth() + 1); return d; }); setSelectedTime(null); };
          const prevMonth = () => { 
            const today = new Date();
            if (viewDate.getMonth() === today.getMonth() && viewDate.getFullYear() === today.getFullYear()) return;
            setViewDate(prev => { const d = new Date(prev); d.setMonth(prev.getMonth() - 1); return d; }); 
            setSelectedTime(null); 
          };

          const handleBooking = () => { if (selectedTime) { setStep(2); setShowModal(true); } };
          const confirmBooking = (e) => { e.preventDefault(); setStep(3); };

          const formattedMonth = lang === 'zh' 
            ? `${viewDate.getFullYear()}${t.yearSuffix} ${viewDate.getMonth() + 1}${t.monthSuffix}`
            : `${viewDate.toLocaleString('en-us', { month: 'long' })} ${viewDate.getFullYear()}`;

          const isCurrentMonth = viewDate.getMonth() === new Date().getMonth() && viewDate.getFullYear() === new Date().getFullYear();

          return (
            <div className="min-h-screen font-sans text-slate-700 relative bg-slate-50 selection:bg-blue-100 selection:text-blue-900 pb-32">
              
              <div className="absolute inset-0 z-0 pointer-events-none overflow-hidden">
                <div className="absolute top-0 right-0 w-[500px] h-[500px] bg-blue-100/40 rounded-full opacity-50 blur-3xl -mr-24 -mt-24"></div>
                <div className="absolute inset-0" style={{ backgroundImage: 'radial-gradient(#CBD5E1 1px, transparent 1px)', backgroundSize: '30px 30px', opacity: 0.15 }}></div>
              </div>

              {/* Navbar */}
              <nav className="relative z-10 sticky top-0 bg-white/95 backdrop-blur-md border-b border-slate-200 px-4 py-4 flex justify-between items-center shadow-sm">
                <div className="flex flex-col">
                  <span className="font-bold text-slate-800 tracking-wide text-sm md:text-base uppercase">{t.navTitle}</span>
                  <span className="text-[10px] text-blue-500 font-bold tracking-wider uppercase">{t.subTitle}</span>
                </div>
                <div className="flex items-center gap-2">
                  <button onClick={toggleLang} className="w-8 h-8 rounded-lg border border-slate-200 text-slate-500 hover:bg-slate-50 transition-colors flex items-center justify-center text-xs font-bold">
                    <Icon name="Globe" size={14} className="mr-0.5" />{lang === 'zh' ? 'EN' : '中'}
                  </button>
                  <a href={LINE_URL} target="_blank" rel="noreferrer" className="bg-blue-600 text-white px-4 py-2 rounded-lg text-xs font-bold tracking-wide hover:bg-blue-700 flex items-center gap-2 shadow-lg shadow-blue-200">
                    <Icon name="MessageCircle" size={14} /><span>{t.consult}</span>
                  </a>
                </div>
              </nav>

              <main className="relative z-10 max-w-lg mx-auto p-4 md:p-6">
                
                {/* 醫師名片 */}
                <div className="bg-white rounded-2xl p-6 shadow-xl shadow-slate-200/50 mb-6 relative border border-slate-100 overflow-hidden">
                    <div className="relative z-10">
                        <h1 className="text-xl font-bold text-slate-800 mb-1">{t.drName}</h1>
                        <p className="text-xs font-bold uppercase tracking-wider text-blue-600 bg-blue-50 inline-block px-2 py-1 rounded mb-4">{t.specialty}</p>
                        <p className="text-sm text-slate-500 leading-relaxed mb-4">{t.description}</p>
                        <div className="flex items-center gap-2 text-xs text-slate-400 border-t border-slate-100 pt-4">
                          <Icon name="Clock" size={14} className="text-blue-500" />
                          <span>{t.hoursLabel}</span>
                        </div>
                    </div>
                    <div className="absolute top-0 right-0 w-32 h-32 bg-blue-50 rounded-full -mr-10 -mt-10 opacity-60"></div>
                </div>

                {/* 月份切換 */}
                <div className="mb-8">
                  <div className="flex items-center justify-between mb-4">
                     <div className="flex items-center gap-4 bg-white px-2 py-1 rounded-full shadow-sm border border-slate-100">
                        <button onClick={prevMonth} disabled={isCurrentMonth} className={`p-1.5 rounded-full ${isCurrentMonth ? 'text-slate-200 cursor-not-allowed' : 'text-slate-600 hover:bg-slate-100'}`}><Icon name="ChevronLeft" size={18} /></button>
                        <span className="font-bold text-slate-800 text-sm min-w-[100px] text-center">{formattedMonth}</span>
                        <button onClick={nextMonth} className="p-1.5 rounded-full text-slate-600 hover:bg-slate-100"><Icon name="ChevronRight" size={18} /></button>
                     </div>
                     <div className="text-xs font-medium text-green-600 flex items-center gap-1 bg-green-50 px-2 py-1 rounded-full">
                        <span className="w-1.5 h-1.5 bg-green-500 rounded-full animate-pulse"></span>Live
                     </div>
                  </div>
                  
                  {/* 日期列與按鈕 */}
                  <div className="relative group">
                    <button onClick={() => scrollDates('left')} className="absolute left-0 top-1/2 -translate-y-1/2 z-20 w-8 h-12 bg-white/80 backdrop-blur-sm border-y border-r border-slate-200 rounded-r-lg flex items-center justify-center text-slate-400 opacity-0 group-hover:opacity-100 transition-opacity"><Icon name="ChevronLeft" size={20} /></button>
                    <div ref={dateScrollRef} className="flex gap-2 overflow-x-auto pb-4 scrollbar-hide active-date-scroll px-1">
                        {availableDates.map((dateObj, idx) => (
                            <button key={idx} disabled={dateObj.isPast} onClick={() => { setSelectedDateIndex(idx); setSelectedTime(null); scrollToIndex(idx); }}
                              className={`flex-1 min-w-[70px] rounded-xl p-3 border transition-all flex flex-col items-center justify-center gap-1 shrink-0 ${selectedDateIndex === idx ? 'bg-blue-600 border-blue-600 text-white shadow-lg shadow-blue-200 scale-105' : dateObj.isPast ? 'bg-slate-50 border-transparent text-slate-200' : 'bg-white border-slate-200 text-slate-500 hover:border-blue-300'}`}
                            >
                            <span className={`text-xs font-medium ${selectedDateIndex === idx ? 'text-blue-100' : ''}`}>{t.dayPrefix}{dateObj.dayName}</span>
                            <span className="text-lg font-bold">{dateObj.label.split('/')[1]}</span>
                            <span className="text-[10px] opacity-60">{dateObj.label}</span>
                            </button>
                        ))}
                    </div>
                    <button onClick={() => scrollDates('right')} className="absolute right-0 top-1/2 -translate-y-1/2 z-20 w-8 h-12 bg-white/80 backdrop-blur-sm border-y border-l border-slate-200 rounded-l-lg flex items-center justify-center text-slate-400 opacity-0 group-hover:opacity-100 transition-opacity"><Icon name="ChevronRight" size={20} /></button>
                  </div>
                </div>

                {/* 時段列表 */}
                <div className="mb-12 fade-in" key={selectedDateIndex}>
                  <h2 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
                    {currentDateObj?.dayIndex === 0 || currentDateObj?.dayIndex === 2 ? t.closedToday : t.selectTime}
                    {currentDateObj?.dayIndex !== 0 && currentDateObj?.dayIndex !== 2 && <span className="text-xs text-slate-400 font-normal">{t.note}</span>}
                  </h2>
                  
                  {currentDateObj?.dayIndex === 0 || currentDateObj?.dayIndex === 2 ? (
                    <div className="flex flex-col items-center justify-center py-16 bg-white rounded-2xl border border-slate-200 text-slate-400">
                       <Icon name="Calendar" size={32} className="mb-4 opacity-30" />
                       <p className="text-lg font-bold text-slate-500">{t.closedToday}</p>
                    </div>
                  ) : currentDaySlots.length > 0 ? (
                    <div className="grid grid-cols-3 md:grid-cols-4 gap-3">
                      {currentDaySlots.map((slot, idx) => (
                        <button key={idx} disabled={slot.status === 'full'} onClick={() => handleTimeSelect(slot)}
                          className={`py-3 px-1 rounded-lg border text-center transition-all ${slot.status === 'full' ? 'bg-slate-100 border-transparent text-slate-300 cursor-not-allowed' : selectedTime === slot.time ? 'bg-blue-600 border-blue-600 text-white shadow-md' : 'bg-white border-slate-200 text-slate-600 hover:border-blue-400'}`}
                        >
                          <span className="font-mono font-bold text-sm block">{slot.time}</span>
                          <span className={`text-[10px] block mt-1 ${selectedTime === slot.time ? 'text-blue-100' : slot.status === 'full' ? 'text-slate-300' : 'text-blue-500'}`}>{slot.status === 'full' ? t.full : t.available}</span>
                        </button>
                      ))}
                    </div>
                  ) : (
                    <div className="flex flex-col items-center justify-center py-16 bg-white rounded-2xl border border-dashed border-slate-200 text-slate-400">
                       <Icon name="Clock" size={32} className="mb-3 opacity-30"/><p className="text-sm font-medium">{t.noSlots}</p>
                    </div>
                  )}
                </div>
              </main>

              {/* 底部按鈕欄 */}
              <div className="fixed bottom-0 left-0 right-0 p-4 md:p-6 bg-white/95 backdrop-blur-md border-t border-slate-200 z-20 shadow-2xl">
                <div className="max-w-lg mx-auto flex items-center justify-between gap-4">
                    <div className="flex flex-col">
                        <span className="text-[10px] text-slate-400 uppercase font-bold tracking-wider">{t.selectedLabel}</span>
                        {selectedTime && currentDateObj ? (
                          <div className="text-lg font-bold text-slate-800">{currentDateObj.label} <span className="text-blue-600">| {selectedTime}</span></div>
                        ) : ( <span className="text-sm text-slate-400">{t.selectPrompt}</span> )}
                    </div>
                    <button disabled={!selectedTime} onClick={handleBooking} className={`px-8 py-3 rounded-xl font-bold text-sm transition-all shadow-lg ${selectedTime ? 'bg-blue-600 text-white hover:bg-blue-700 active:scale-95' : 'bg-slate-200 text-slate-400 cursor-not-allowed'}`}>{t.bookBtn}</button>
                </div>
              </div>

              {/* 彈窗系統 */}
              {showModal && (
                <div className="fixed inset-0 z-50 flex items-center justify-center p-6 bg-slate-900/50 backdrop-blur-sm" onClick={() => setShowModal(false)}>
                  <div className="bg-white w-full max-w-sm rounded-2xl shadow-2xl relative animate-in fade-in zoom-in duration-200 overflow-hidden max-h-[90vh] overflow-y-auto" onClick={e => e.stopPropagation()}>
                    <div className="h-2 bg-blue-600 w-full sticky top-0 z-20"></div>
                    {step === 2 ? (
                      <div className="p-6 md:p-8">
                        <h3 className="text-xl font-bold text-slate-800 mb-6 text-center">{t.modalTitle}</h3>
                        <div className="text-center mb-6"><div className="inline-block bg-blue-50 text-blue-700 px-4 py-2 rounded-full text-sm font-bold border border-blue-100">{currentDateObj?.label} • {selectedTime}</div></div>
                        <form onSubmit={confirmBooking} className="space-y-4">
                          <div className="space-y-1"><label className="text-xs font-bold text-slate-500 uppercase">{t.labelName}</label><input required type="text" placeholder={t.placeholderName} className="w-full bg-slate-50 border border-slate-200 rounded-lg px-4 py-3 text-sm focus:ring-2 focus:ring-blue-500/20 outline-none" /></div>
                          <div className="space-y-1"><label className="text-xs font-bold text-slate-500 uppercase">{t.labelPhone}</label><input required type="tel" placeholder="09xx-xxx-xxx" className="w-full bg-slate-50 border border-slate-200 rounded-lg px-4 py-3 text-sm focus:ring-2 focus:ring-blue-500/20 outline-none" /></div>
                          <div className="space-y-2">
                            <label className="text-xs font-bold text-slate-500 uppercase">{t.labelService}</label>
                            <div className="grid grid-cols-2 gap-2">
                                {SERVICE_OPTIONS.map(opt => (
                                    <button key={opt.id} type="button" onClick={() => handleServiceToggle(opt.id)} className={`text-[10px] md:text-xs py-2 px-2 rounded-lg border transition-all flex items-center gap-2 ${selectedServices.includes(opt.id) ? 'bg-blue-50 border-blue-500 text-blue-700 font-bold' : 'bg-slate-50 border-slate-200 text-slate-600'}`}>
                                        <div className={`w-3 h-3 rounded border flex items-center justify-center shrink-0 ${selectedServices.includes(opt.id) ? 'bg-blue-500 border-blue-500' : 'bg-white border-slate-300'}`}>{selectedServices.includes(opt.id) && <Icon name="Check" size={10} className="text-white" />}</div>
                                        <span>{lang === 'zh' ? opt.zh : opt.en}</span>
                                    </button>
                                ))}
                            </div>
                          </div>
                          <div className="pt-4 space-y-3">
                            <button type="submit" className="w-full bg-blue-600 text-white font-bold py-4 rounded-xl hover:bg-blue-700 shadow-lg shadow-blue-200 active:scale-95 transition-transform">{t.confirmBtn}</button>
                            <button type="button" onClick={() => setShowModal(false)} className="w-full text-slate-400 text-sm font-bold py-2 hover:text-slate-600">{t.cancelBtn}</button>
                          </div>
                        </form>
                      </div>
                    ) : (
                      <div className="p-10 text-center">
                        <div className="w-16 h-16 bg-green-100 text-green-500 rounded-full flex items-center justify-center mx-auto mb-6"><Icon name="CheckCircle" size={32} /></div>
                        <h3 className="text-2xl font-bold text-slate-800 mb-4">{t.successTitle}</h3>
                        <p className="text-slate-500 text-sm mb-8 leading-relaxed whitespace-pre-line">{t.successDesc}</p>
                        <div className="bg-slate-50 p-4 rounded-xl border border-dashed border-slate-200 mb-8"><div className="text-xs text-slate-400 font-bold mb-1">{t.lineIdLabel}</div><div className="font-mono text-xl font-bold text-slate-700 select-all">{LINE_ID}</div></div>
                        <a href={LINE_URL} target="_blank" rel="noreferrer" className="block w-full bg-[#06C755] text-white font-bold py-4 rounded-xl hover:bg-[#05b34c] flex items-center justify-center gap-2 shadow-lg shadow-green-100"><Icon name="MessageCircle" size={18} />{t.addLineBtn}</a>
                        <button onClick={() => {setShowModal(false); setStep(1); setSelectedServices([]); setSelectedTime(null);}} className="mt-6 text-xs text-slate-400 font-bold hover:text-slate-600">{t.closeBtn}</button>
                      </div>
                    )}
                  </div>
                </div>
              )}
            </div>
          );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<MedicalBookingApp />);
    </script>
</body>
</html>
