<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FOREX SIGNAL AI</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        dark: '#08090a',
                    }
                }
            }
        }
    </script>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body class="bg-[#08090a] text-white">

<div id="root"></div>

<script type="text/babel">
const { useState, useEffect } = React;

const App = () => {
    const forexAssets = [
        { name: 'EUR/USD (OTC)', payout: '92%' },
        { name: 'GBP/USD (OTC)', payout: '92%' },
        { name: 'USD/JPY (OTC)', payout: '91%' },
        { name: 'EUR/GBP (OTC)', payout: '89%' },
        { name: 'AUD/USD (OTC)', payout: '88%' },
        { name: 'USD/CAD (OTC)', payout: '87%' },
        { name: 'EUR/JPY (OTC)', payout: '90%' },
        { name: 'NZD/USD (OTC)', payout: '85%' },
        { name: 'GBP/JPY (OTC)', payout: '89%' },
        { name: 'CHF/JPY (OTC)', payout: '82%' },
        { name: 'EUR/CHF (OTC)', payout: '85%' },
        { name: 'AUD/CAD (OTC)', payout: '88%' }
    ];

    const [selectedAsset, setSelectedAsset] = useState('EUR/USD (OTC)');
    const [price, setPrice] = useState(1.08542);
    const [isRunning, setIsRunning] = useState(false);
    const [rsi, setRsi] = useState(50);
    const [signal, setSignal] = useState(null);
    const [history, setHistory] = useState([]);
    const [countdown, setCountdown] = useState(0); 
    const [isLocked, setIsLocked] = useState(false);

    // --- دالة حساب قوة الإشارة ---
    const calculateSignalStrength = (price, rsi) => {
        const rsiScore = rsi < 28 ? 30 : rsi > 72 ? 30 : 15;
        const smaScore = Math.random() * 25;
        const macdScore = Math.random() * 25;
        const bollScore = Math.random() * 20;
        const total = rsiScore + smaScore + macdScore + bollScore;
        return Math.min(100, Math.max(0, Math.round(total)));
    };

    const triggerSignal = (title, type, entry) => {
        const strength = calculateSignalStrength(entry, rsi);
        const newSig = {
            id: Date.now(),
            title,
            type,
            asset: selectedAsset,
            price: entry.toFixed(5),
            time: new Date().toLocaleTimeString('ar-EG'),
            expiry: '60s',
            strength
        };
        setSignal(newSig);
        setHistory(prev => [newSig, ...prev].slice(0, 10));
        setCountdown(60); 
        setIsLocked(true); 
        
        if (navigator.vibrate) navigator.vibrate([200, 100, 200]);
    };

    useEffect(() => {
        let interval;
        if (isRunning) {
            interval = setInterval(() => {
                setPrice(prev => {
                    const volatility = 0.00015;
                    const move = (Math.random() - 0.5) * volatility;
                    const newPrice = prev + move;

                    const newRsi = Math.max(10, Math.min(90, rsi + (Math.random() - 0.5) * 12));
                    setRsi(newRsi);

                    if (!isLocked) {
                        if (newRsi < 28) triggerSignal('شراء (CALL)', 'CALL', newPrice);
                        else if (newRsi > 72) triggerSignal('بيع (PUT)', 'PUT', newPrice);
                    }
                    return newPrice;
                });
            }, 500);
        }
        return () => clearInterval(interval);
    }, [isRunning, rsi, isLocked]);

    useEffect(() => {
        let timer;
        if (countdown > 0) timer = setInterval(() => setCountdown(prev => prev - 1), 1000);
        else if (countdown === 0 && isLocked) {
            setIsLocked(false);
            setSignal(null);
        }
        return () => clearInterval(timer);
    }, [countdown, isLocked]);

    return (
        <div className="min-h-screen bg-[#08090a] text-white font-sans p-4 pb-24 select-none overflow-x-hidden">
            {/* Header */}
            <div className="flex justify-between items-center mb-6">
                <div className="flex items-center gap-3">
                    <div className="w-10 h-10 bg-blue-600 rounded-xl flex items-center justify-center shadow-lg shadow-blue-600/20 border border-blue-400/20">🌐</div>
                    <div>
                        <h1 className="text-lg font-black tracking-tight leading-none">FOREX SIGNAL AI</h1>
                        <p className="text-[9px] text-blue-400 font-bold uppercase tracking-widest mt-1">Pocket Option / IQ Option</p>
                    </div>
                </div>
                <div className={`px-3 py-1.5 rounded-full text-[9px] font-black border uppercase transition-all duration-500 ${isLocked ? 'bg-red-500/10 border-red-500/40 text-red-500' : 'bg-green-500/10 border-green-500/40 text-green-500'}`}>
                   {isLocked ? 'جارٍ تنفيذ الصفقة' : 'البحث عن فرصة'}
                </div>
            </div>

            {/* Selector */}
            <div className="bg-[#111216] p-4 rounded-3xl border border-white/5 mb-4 shadow-inner">
                <label className="text-[10px] text-gray-500 font-black uppercase mb-3 block px-1">أزواج العملات المتاحة</label>
                <select 
                    value={selectedAsset} 
                    onChange={(e) => {setSelectedAsset(e.target.value); setSignal(null);}}
                    disabled={isLocked}
                    className="w-full bg-[#1a1b21] text-sm font-bold p-4 rounded-2xl border border-white/5 outline-none focus:border-blue-500/50 transition-all disabled:opacity-50 appearance-none cursor-pointer"
                >
                    {forexAssets.map(a => <option key={a.name} value={a.name}>{a.name} - ربح {a.payout}</option>)}
                </select>
            </div>

            {/* Main Display */}
            <div className="bg-gradient-to-br from-[#121318] to-[#08090a] rounded-[2.5rem] p-8 border border-white/5 mb-6 text-center shadow-2xl relative overflow-hidden">
                <div className="absolute top-0 right-0 p-8 opacity-5">📊</div>
                <p className="text-[10px] text-gray-500 uppercase tracking-[0.3em] mb-3 font-black opacity-70 italic">Live Quote Market</p>
                <h2 className="text-6xl font-mono font-black tracking-tighter mb-8 bg-clip-text text-transparent bg-gradient-to-b from-white to-gray-500">
                    {price.toFixed(5)}
                </h2>

                {/* RSI Meter */}
                <div className="space-y-2">
                    <div className="flex justify-between text-[8px] font-black text-gray-600 uppercase italic">
                        <span>تشبع بيعي</span>
                        <span>توازن السعر</span>
                        <span>تشبع شرائي</span>
                    </div>
                    <div className="h-2 bg-black/50 rounded-full p-0.5 border border-white/5">
                        <div className={`h-full rounded-full transition-all duration-700 shadow-sm ${rsi > 72 ? 'bg-red-500' : rsi < 28 ? 'bg-green-500' : 'bg-blue-600'}`}
                            style={{width: `${rsi}%`}}></div>
                    </div>
                </div>
            </div>

            {/* Dynamic Action */}
            {!isRunning ? (
                <button onClick={() => setIsRunning(true)}
                    className="w-full py-5 rounded-[2rem] bg-blue-600 text-white font-black text-xl shadow-xl shadow-blue-600/30 active:scale-95 transition-all flex items-center justify-center gap-3 border-t border-blue-400/30">▶️ ابدأ الفحص الذكي</button>
            ) : (
                <div className="space-y-4">
                    {signal ? (
                        <div className={`p-6 rounded-[2.5rem] border-2 transition-all duration-500 ${signal.type === 'CALL' ? 'bg-green-500/10 border-green-500/50 shadow-lg shadow-green-500/5' : 'bg-red-500/10 border-red-500/50 shadow-lg shadow-red-500/5'}`}>
                            <div className="flex justify-between items-center mb-6">
                                <div className="flex items-center gap-4">
                                    <div className={`w-16 h-16 rounded-2xl flex items-center justify-center shadow-2xl ${signal.type==='CALL'?'bg-green-600':'bg-red-600'}`}>{signal.type==='CALL'?'📈':'📉'}</div>
                                    <div>
                                        <h4 className={`text-2xl font-black italic tracking-tight ${signal.type==='CALL'?'text-green-500':'text-red-500'}`}>{signal.title}</h4>
                                        <p className="text-[10px] text-gray-400 font-black uppercase tracking-widest">{signal.asset}</p>
                                        <p className="text-[10px] mt-2 font-bold">قوة الإشارة: {signal.strength}% {signal.strength>=80?'⚡':signal.strength>=70?'🔹':'⚪'}</p>
                                    </div>
                                </div>
                                <div className="relative w-16 h-16 flex items-center justify-center">
                                    <svg className="absolute inset-0 w-full h-full -rotate-90">
                                        <circle cx="32" cy="32" r="28" stroke="currentColor" strokeWidth="4" fill="transparent" className="text-white/10" />
                                        <circle cx="32" cy="32" r="28" stroke="currentColor" strokeWidth="4" fill="transparent"
                                            className={signal.type==='CALL'?'text-green-500':'text-red-500'}
                                            strokeDasharray="176"
                                            strokeDashoffset={176-(176*countdown)/60}/>
                                    </svg>
                                    <span className="text-xl font-mono font-black">{countdown}</span>
                                </div>
                            </div>
                        </div>
                    ) : (
                        <div className="p-12 border border-dashed border-white/10 rounded-[2.5rem] text-center flex flex-col items-center gap-4 bg-[#0a0b0f] shadow-inner">
                            🔄
                            <p className="text-[10px] text-zinc-500 font-black italic tracking-widest uppercase animate-pulse">جاري تحليل زخم {selectedAsset}...</p>
                        </div>
                    )}

                    <button onClick={() => {setIsRunning(false); setSignal(null); setCountdown(0); setIsLocked(false);}}
                        className="w-full py-4 rounded-2xl bg-[#111216] text-zinc-600 font-black text-[10px] uppercase tracking-[0.3em] border border-white/5 active:bg-zinc-900">
                        إيقاف المحلل
                    </button>
                </div>
            )}

            {/* History */}
            <div className="mt-10">
                <div className="flex justify-between items-center mb-4 px-2">
                    <h3 className="text-[10px] font-black text-gray-600 uppercase tracking-[0.2em] flex items-center gap-2">📊 سجل إشارات المنصة</h3>
                    <span className="text-[8px] text-blue-500 font-black bg-blue-500/10 px-2 py-0.5 rounded">LIVE LOG</span>
                </div>
                <div className="space-y-2">
                    {history.map(h => (
                        <div key={h.id} className="bg-[#111216] p-4 rounded-2xl border border-white/5 flex justify-between items-center transition-all hover:bg-[#1a1b21]">
                            <div className="flex items-center gap-3">
                                <div className={`p-2 rounded-lg ${h.type==='CALL'?'bg-green-500/10 text-green-500':'bg-red-500/10 text-red-500'}`}>
                                    {h.type==='CALL'?'📈':'📉'}
                                </div>
                                <div>
                                    <p className="text-[11px] font-black tracking-tight">{h.asset}</p>
                                    <p className="text-[8px] text-gray-600 font-bold uppercase">{h.time}</p>
                                </div>
                            </div>
                            <div className="text-left font-mono">
                                <p className="text-xs font-black">{h.price}</p>
                                <p className="text-[8px] text-green-500 font-black uppercase tracking-tighter">Completed</p>
                            </div>
                        </div>
                    ))}
                    {history.length===0 && (
                        <div className="text-center py-6 text-zinc-800 italic text-[10px] font-bold uppercase tracking-widest border border-dashed border-zinc-900 rounded-3xl">لا يوجد سجل حالي</div>
                    )}
                </div>
            </div>
        </div>
    );
};

const container = document.getElementById('root');
const root = ReactDOM.createRoot(container);
root.render(<App />);
</script>
</body>
</html>
