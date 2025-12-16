import React, { useState, useEffect } from 'react';
import { AreaChart, Area, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, ReferenceDot, ReferenceLine, Label } from 'recharts';
import { TrendingDown, TrendingUp, AlertTriangle, ShieldAlert, Cpu, Crown, Handshake, Rocket, DollarSign, Activity } from 'lucide-react';

// 模擬數據：結合歷史走勢與虛構的未來預測
// 使用數值型 yearValue 進行 X 軸定位 (例如 2020.58 代表 2020年7月)
const data = [
  { year: '2007', yearValue: 2007, price: 28, label: '' },
  { year: '2008', yearValue: 2008, price: 15, label: '' },
  { year: '2009', yearValue: 2009.1, price: 12.5, eventType: 'crash', title: '金融海嘯低點', desc: '股價觸及約 $12' },
  { year: '2012', yearValue: 2012, price: 25, label: '' },
  { year: '2015', yearValue: 2015, price: 35, label: '' },
  { year: '2018', yearValue: 2018.5, price: 55, eventType: 'warning', title: '資安漏洞 & CEO 辭職', desc: 'Spectre/Meltdown 陰影' },
  { year: '2019', yearValue: 2019, price: 50, label: '' },
  { year: '2020.07', yearValue: 2020.58, price: 48, eventType: 'drop', title: '7nm 製程延遲', desc: '單日暴跌 16%，技術落後隱憂' },
  { year: '2021', yearValue: 2021.1, price: 68, eventType: 'peak', title: 'Gelsinger 回歸', desc: 'IDM 2.0 戰略，創近年新高' },
  { year: '2022', yearValue: 2022, price: 45, label: '' },
  { year: '2023', yearValue: 2023, price: 35, label: '' },
  { year: '2024.01', yearValue: 2024.1, price: 48, label: '' },
  { year: '2024.08', yearValue: 2024.6, price: 20, eventType: 'disaster', title: '暫停配息/財報重挫', desc: '裁員 1.5 萬人，股價崩跌至十年低點', isFutureStart: true },
  { year: '2024.12', yearValue: 2024.9, price: 22, label: '築底' },
  { year: '2025.03', yearValue: 2025.2, price: 26, eventType: 'recovery', title: '陳立武 (Lip-Bu Tan) 出任 CEO', desc: '半導體老將回歸，改革訊號' },
  { year: '2025.08', yearValue: 2025.6, price: 34, eventType: 'invest', title: '美國政府 & 軟銀入股', desc: 'CHIPS Act 注資 & 軟銀 $20億投資' },
  { year: '2025.09', yearValue: 2025.75, price: 42, eventType: 'rocket', title: 'Nvidia 入股與戰略合作', desc: 'IFS 代工業務獲最大背書，目標 $40+' },
];

// 自定義圖標渲染組件
const CustomDot = (props) => {
  const { cx, cy, payload, onEventClick, activeEvent } = props;
  
  if (!payload.eventType) return null;

  let IconComponent = Activity;
  let color = "#ffffff";
  let glowColor = "rgba(255,255,255,0.5)";

  switch (payload.eventType) {
    case 'crash':
      IconComponent = TrendingDown;
      color = "#ef4444"; // Red
      glowColor = "rgba(239, 68, 68, 0.6)";
      break;
    case 'warning':
      IconComponent = ShieldAlert;
      color = "#f97316"; // Orange
      glowColor = "rgba(249, 115, 22, 0.6)";
      break;
    case 'drop':
      IconComponent = AlertTriangle;
      color = "#ef4444";
      glowColor = "rgba(239, 68, 68, 0.6)";
      break;
    case 'peak':
      IconComponent = Crown;
      color = "#eab308"; // Gold
      glowColor = "rgba(234, 179, 8, 0.6)";
      break;
    case 'disaster':
      IconComponent = Activity; // Using Activity as "Broken" metaphor
      color = "#dc2626"; // Dark Red
      glowColor = "rgba(220, 38, 38, 0.8)";
      break;
    case 'recovery':
      IconComponent = Handshake;
      color = "#22c55e"; // Green
      glowColor = "rgba(34, 197, 94, 0.6)";
      break;
    case 'invest':
      IconComponent = DollarSign;
      color = "#3b82f6"; // Blue
      glowColor = "rgba(59, 130, 246, 0.6)";
      break;
    case 'rocket':
      IconComponent = Rocket;
      color = "#a855f7"; // Purple/Neon
      glowColor = "rgba(168, 85, 247, 0.8)";
      break;
    default:
      break;
  }

  const isActive = activeEvent && activeEvent.yearValue === payload.yearValue;
  const size = isActive ? 32 : 24;

  return (
    <svg 
      x={cx - size / 2} 
      y={cy - size / 2} 
      width={size} 
      height={size} 
      viewBox="0 0 24 24"
      className={`cursor-pointer transition-all duration-300 ${isActive ? 'scale-125 z-50' : 'hover:scale-110'}`}
      onClick={() => onEventClick(payload)}
      style={{ filter: `drop-shadow(0 0 8px ${glowColor})` }}
    >
      <circle cx="12" cy="12" r="12" fill="#0f172a" stroke={color} strokeWidth="2" />
      <IconComponent size={14} color={color} x="5" y="5" />
    </svg>
  );
};

// 資訊卡片組件
const EventCard = ({ event, onClose }) => {
  if (!event) return null;

  let borderColor = "border-gray-500";
  let iconColor = "text-gray-400";
  let Icon = Activity;

  switch (event.eventType) {
    case 'crash': case 'drop': case 'disaster':
      borderColor = "border-red-500";
      iconColor = "text-red-500";
      Icon = TrendingDown;
      break;
    case 'warning':
      borderColor = "border-orange-500";
      iconColor = "text-orange-500";
      Icon = ShieldAlert;
      break;
    case 'peak':
      borderColor = "border-yellow-500";
      iconColor = "text-yellow-400";
      Icon = Crown;
      break;
    case 'recovery':
      borderColor = "border-green-500";
      iconColor = "text-green-400";
      Icon = Handshake;
      break;
    case 'invest':
      borderColor = "border-blue-500";
      iconColor = "text-blue-400";
      Icon = DollarSign;
      break;
    case 'rocket':
      borderColor = "border-purple-500";
      iconColor = "text-purple-400";
      Icon = Rocket;
      break;
    default: break;
  }

  return (
    <div className={`absolute z-50 p-4 rounded-xl border ${borderColor} bg-slate-900/95 backdrop-blur-md shadow-2xl text-white max-w-xs animate-in fade-in zoom-in duration-300 top-20 left-1/2 transform -translate-x-1/2 md:left-auto md:translate-x-0 md:top-24 md:right-10`}>
      <div className="flex items-start gap-3">
        <div className={`p-2 rounded-full bg-slate-800 ${iconColor} shadow-[0_0_15px_rgba(0,0,0,0.5)]`}>
          <Icon size={24} />
        </div>
        <div>
          <h3 className="font-bold text-lg leading-tight mb-1 text-cyan-50">{event.title}</h3>
          <p className="text-xs text-cyan-300 font-mono mb-2">{event.year}</p>
          <p className="text-sm text-slate-300 leading-relaxed">{event.desc}</p>
          <div className="mt-3 flex items-center gap-2">
            <span className="text-xs px-2 py-1 rounded bg-slate-800 border border-slate-700 text-slate-400">
              股價: ${event.price}
            </span>
            {event.eventType === 'rocket' && (
              <span className="text-xs px-2 py-1 rounded bg-purple-900/30 border border-purple-500/50 text-purple-300 flex items-center gap-1">
                <Cpu size={10} /> Nvidia Inside
              </span>
            )}
          </div>
        </div>
      </div>
      <button 
        onClick={onClose}
        className="absolute top-2 right-2 text-slate-500 hover:text-white"
      >
        ×
      </button>
    </div>
  );
};

export default function IntelStockChart() {
  const [activeEvent, setActiveEvent] = useState(null);
  
  // 預設選中最新的事件
  useEffect(() => {
    setActiveEvent(data[data.length - 1]);
  }, []);

  return (
    <div className="w-full min-h-screen bg-slate-950 flex flex-col items-center justify-center p-4 relative overflow-hidden font-sans">
      
      {/* 背景電路板效果 */}
      <div className="absolute inset-0 z-0 opacity-20 pointer-events-none" 
        style={{
            backgroundImage: `
                radial-gradient(circle at 50% 50%, rgba(6, 182, 212, 0.1) 1px, transparent 1px),
                linear-gradient(rgba(6, 182, 212, 0.05) 1px, transparent 1px),
                linear-gradient(90deg, rgba(6, 182, 212, 0.05) 1px, transparent 1px)
            `,
            backgroundSize: '40px 40px, 20px 20px, 20px 20px'
        }}
      ></div>
      
      {/* 標題區域 */}
      <div className="relative z-10 w-full max-w-6xl mb-6 flex flex-col md:flex-row justify-between items-end border-b border-slate-800 pb-4">
        <div>
            <div className="flex items-center gap-2 mb-1">
                <Cpu className="text-cyan-400" />
                <span className="text-cyan-400 font-mono text-sm tracking-widest">INTC STOCK ANALYSIS</span>
            </div>
            <h1 className="text-3xl md:text-5xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-cyan-300 via-blue-200 to-white filter drop-shadow-[0_0_10px_rgba(6,182,212,0.5)]">
                INTEL 重生之路
            </h1>
            <p className="text-slate-400 mt-2 text-sm md:text-base">
                2007 - 2025 歷史回顧與未來戰略預測
            </p>
        </div>
        <div className="text-right mt-4 md:mt-0">
            <div className="text-3xl font-mono text-green-400 font-bold flex items-center justify-end gap-2">
                $42.00 <span className="text-xs text-slate-500 bg-slate-900 px-1 border border-slate-800 rounded">TARGET</span>
            </div>
            <div className="text-xs text-slate-500 uppercase tracking-wider">2025 Q3 Forecast</div>
        </div>
      </div>

      {/* 圖表主要區域 */}
      <div className="relative w-full max-w-6xl h-[500px] bg-slate-900/50 rounded-2xl border border-slate-800 shadow-[0_0_50px_rgba(0,0,0,0.5)] backdrop-blur-sm p-4">
        
        {/* 左上角圖例 */}
        <div className="absolute top-6 left-6 z-10 flex gap-4 text-xs">
            <div className="flex items-center gap-2">
                <div className="w-3 h-3 rounded-full bg-cyan-500 shadow-[0_0_8px_rgba(6,182,212,1)]"></div>
                <span className="text-slate-300">歷史走勢</span>
            </div>
            <div className="flex items-center gap-2">
                <div className="w-3 h-3 rounded-full bg-green-400 shadow-[0_0_8px_rgba(74,222,128,1)] animate-pulse"></div>
                <span className="text-green-300 font-bold">未來預測 (重生篇章)</span>
            </div>
        </div>

        <ResponsiveContainer width="100%" height="100%">
          <AreaChart data={data} margin={{ top: 20, right: 30, left: 0, bottom: 20 }}>
            <defs>
              <linearGradient id="colorPrice" x1="0" y1="0" x2="1" y2="0">
                <stop offset="0%" stopColor="#06b6d4" stopOpacity={0.8}/>
                <stop offset="80%" stopColor="#06b6d4" stopOpacity={0.3}/>
                <stop offset="80%" stopColor="#22c55e" stopOpacity={0.8}/>
                <stop offset="100%" stopColor="#22c55e" stopOpacity={0.9}/>
              </linearGradient>
              <linearGradient id="fillGradient" x1="0" y1="0" x2="0" y2="1">
                <stop offset="5%" stopColor="#06b6d4" stopOpacity={0.2}/>
                <stop offset="95%" stopColor="#06b6d4" stopOpacity={0}/>
              </linearGradient>
              <filter id="glow" height="200%" width="200%">
                <feGaussianBlur in="SourceGraphic" stdDeviation="3" result="blur" />
                <feMerge>
                    <feMergeNode in="blur" />
                    <feMergeNode in="SourceGraphic" />
                </feMerge>
              </filter>
            </defs>
            
            <CartesianGrid strokeDasharray="3 3" stroke="#1e293b" vertical={false} />
            
            <XAxis 
                dataKey="yearValue" 
                type="number" 
                domain={[2007, 2026]} 
                tick={{ fill: '#64748b', fontSize: 12 }} 
                tickCount={10}
                tickFormatter={(value) => Math.floor(value)}
                axisLine={false}
            />
            
            <YAxis 
                domain={[0, 80]} 
                tick={{ fill: '#64748b', fontSize: 12 }} 
                tickFormatter={(value) => `$${value}`}
                axisLine={false}
                tickLine={false}
            />
            
            <Tooltip 
                contentStyle={{ backgroundColor: '#0f172a', border: '1px solid #1e293b', borderRadius: '8px' }}
                itemStyle={{ color: '#22d3ee' }}
                labelStyle={{ color: '#94a3b8' }}
                formatter={(value) => [`$${value}`, '股價']}
                labelFormatter={(value) => `年份: ${Math.floor(value)}`}
            />

            {/* 2024.08 分界線 */}
            <ReferenceLine x={2024.6} stroke="#ef4444" strokeDasharray="3 3" strokeOpacity={0.5}>
                <Label value="現今" position="insideTopLeft" fill="#ef4444" fontSize={12} offset={10} />
            </ReferenceLine>

            <Area 
                type="monotone" 
                dataKey="price" 
                stroke="url(#colorPrice)" 
                strokeWidth={3}
                fill="url(#fillGradient)" 
                filter="url(#glow)"
                animationDuration={2000}
                dot={<CustomDot onEventClick={setActiveEvent} activeEvent={activeEvent} />}
            />

          </AreaChart>
        </ResponsiveContainer>

        {/* 浮動的事件詳情卡片 (固定位置或隨選中出現) */}
        <EventCard event={activeEvent} onClose={() => setActiveEvent(null)} />

      </div>

      {/* 底部時間軸導航輔助 (手機版更友善) */}
      <div className="w-full max-w-6xl mt-6 grid grid-cols-4 md:grid-cols-7 gap-2">
        {data.filter(d => d.eventType).map((d, idx) => (
            <button 
                key={idx}
                onClick={() => setActiveEvent(d)}
                className={`text-xs p-2 rounded border transition-all ${
                    activeEvent && activeEvent.yearValue === d.yearValue 
                    ? 'bg-cyan-900/50 border-cyan-500 text-cyan-300 shadow-[0_0_10px_rgba(6,182,212,0.3)]' 
                    : 'bg-slate-900 border-slate-800 text-slate-500 hover:border-slate-600'
                }`}
            >
                <div className="font-mono mb-1">{d.year}</div>
                <div className="truncate text-[10px]">{d.title.split(' ')[0]}</div>
            </button>
        ))}
      </div>

    </div>
  );
}
