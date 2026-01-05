import React, { useState } from 'react';
import { Calculator, Box, Columns, Layers, Info, Square, Droplets, ShieldCheck, Copyright } from 'lucide-react';

const App = () => {
  const [activeTab, setActiveTab] = useState('footing');
  const [concreteGrade, setConcreteGrade] = useState('25'); 
  
  // States for building components
  const [footing, setFooting] = useState({ w: 1.2, l: 1.2, d: 0.4, qty: 1, rebarSize: 12, spacing: 150, layers: 1 });
  const [slab, setSlab] = useState({ w: 4, l: 5, d: 0.12, qty: 1, rebarSize: 10, spacing: 150, layers: 1 });
  const [column, setColumn] = useState({ w: 0.25, l: 0.25, h: 3.5, qty: 1, mainRebar: 4, mainSize: 12, stirrupSize: 6, stirrupSpacing: 150 });
  const [beam, setBeam] = useState({ w: 0.2, h: 0.4, l: 4, qty: 1, mainTop: 2, mainBottom: 2, mainSize: 14, stirrupSize: 6, stirrupSpacing: 150 });

  const mixRatios = {
    '15': { cement: 280, sand: 0.48, stone: 0.85, water: 185 },
    '20': { cement: 320, sand: 0.45, stone: 0.85, water: 190 },
    '25': { cement: 380, sand: 0.43, stone: 0.83, water: 195 },
    '30': { cement: 420, sand: 0.41, stone: 0.81, water: 200 }
  };

  const calculateVolume = () => {
    switch(activeTab) {
      case 'footing': return footing.w * footing.l * footing.d * footing.qty;
      case 'slab': return slab.w * slab.l * slab.d * slab.qty;
      case 'column': return column.w * column.l * column.h * column.qty;
      case 'beam': return beam.w * beam.h * beam.l * beam.qty;
      default: return 0;
    }
  };

  const getWeightPerMeter = (size) => (size * size) / 162;

  const calculateSteel = () => {
    let totalKg = 0;
    if (activeTab === 'footing') {
      const barsW = (footing.w * 1000 / footing.spacing) + 1;
      const barsL = (footing.l * 1000 / footing.spacing) + 1;
      const length = (barsW * footing.l + barsL * footing.w) * footing.qty * footing.layers;
      totalKg = length * getWeightPerMeter(footing.rebarSize);
    } else if (activeTab === 'slab') {
      const barsW = (slab.w * 1000 / slab.spacing) + 1;
      const barsL = (slab.l * 1000 / slab.spacing) + 1;
      const length = (barsW * slab.l + barsL * slab.w) * slab.qty * slab.layers;
      totalKg = length * getWeightPerMeter(slab.rebarSize);
    } else if (activeTab === 'column') {
      const main = column.h * column.mainRebar * column.qty * getWeightPerMeter(column.mainSize);
      const stirrupQty = (column.h * 1000 / column.stirrupSpacing) + 1;
      const stirrupLen = ((column.w + column.l) * 2 + 0.15) * stirrupQty * column.qty;
      totalKg = main + (stirrupLen * getWeightPerMeter(column.stirrupSize));
    } else if (activeTab === 'beam') {
      const main = beam.l * (beam.mainTop + beam.mainBottom) * beam.qty * getWeightPerMeter(beam.mainSize);
      const stirrupQty = (beam.l * 1000 / beam.stirrupSpacing) + 1;
      const stirrupLen = ((beam.w + beam.h) * 2 + 0.15) * stirrupQty * beam.qty;
      totalKg = main + (stirrupLen * getWeightPerMeter(beam.stirrupSize));
    }
    return totalKg.toFixed(2);
  };

  const volume = calculateVolume();
  const steel = calculateSteel();
  const mix = mixRatios[concreteGrade];

  const Input = ({ label, value, onChange, unit }) => (
    <div className="mb-4">
      <label className="text-[11px] font-bold text-slate-500 block mb-1 ml-1">{label}</label>
      <div className="relative group">
        <input 
          type="number" 
          value={value} 
          onChange={(e) => onChange(parseFloat(e.target.value) || 0)}
          className="w-full bg-slate-50 border border-slate-200 rounded-2xl p-3 text-sm focus:ring-2 focus:ring-blue-500 transition-all outline-none font-semibold group-hover:border-blue-200 shadow-sm"
        />
        <span className="absolute right-4 top-3 text-[10px] text-slate-400 font-bold uppercase">{unit}</span>
      </div>
    </div>
  );

  return (
    <div className="min-h-screen bg-slate-100 p-2 sm:p-6 font-sans flex flex-col items-center">
      <div className="w-full max-w-md bg-white shadow-2xl rounded-[3rem] overflow-hidden border border-white">
        
        {/* Header Section */}
        <div className="bg-slate-900 p-8 text-white relative">
          <div className="absolute top-4 right-6 flex items-center gap-1 opacity-50">
            <ShieldCheck size={14} className="text-blue-400" />
            <span className="text-[9px] font-bold uppercase tracking-tighter">Verified License</span>
          </div>
          <p className="text-blue-400 text-[10px] uppercase tracking-[0.3em] font-black mb-1">Nem Linda</p>
          <h1 className="text-xl font-bold leading-tight tracking-tight">
            គណនារកបរិមាណដែក និងបេតុងនៃគ្រឿងបង្គុំ
          </h1>
        </div>

        {/* Concrete Grade Selection */}
        <div className="px-8 py-5 bg-slate-50 border-b border-slate-100">
          <p className="text-[10px] font-bold text-slate-400 mb-3 text-center uppercase tracking-widest">ជ្រើសរើសកម្លាំងបេតុង (MPa)</p>
          <div className="grid grid-cols-4 gap-2">
            {['15', '20', '25', '30'].map(g => (
              <button 
                key={g}
                onClick={() => setConcreteGrade(g)}
                className={`py-3 rounded-2xl text-xs font-black transition-all ${concreteGrade === g ? 'bg-blue-600 text-white shadow-lg' : 'bg-white text-slate-400 border border-slate-200'}`}
              >
                {g} MPa
              </button>
            ))}
          </div>
        </div>

        {/* Tab Navigation */}
        <div className="flex bg-white px-2 border-b border-slate-50">
          {[
            { id: 'footing', icon: Box, label: 'ជើងតាង' },
            { id: 'slab', icon: Square, label: 'ប្លង់សេ' },
            { id: 'column', icon: Columns, label: 'សសរ' },
            { id: 'beam', icon: Layers, label: 'ធ្នឹម' }
          ].map(tab => (
            <button
              key={tab.id}
              onClick={() => setActiveTab(tab.id)}
              className={`flex-1 py-5 flex flex-col items-center gap-1.5 transition-all relative ${activeTab === tab.id ? 'text-blue-600' : 'text-slate-300'}`}
            >
              <tab.icon size={22} />
              <span className="text-[10px] font-black uppercase tracking-widest">{tab.label}</span>
              {activeTab === tab.id && <div className="absolute bottom-0 w-8 h-1 bg-blue-600 rounded-full" />}
            </button>
          ))}
        </div>

        {/* Input Forms */}
        <div className="p-8 bg-white min-h-[320px]">
          <div className="grid grid-cols-2 gap-x-4">
            {activeTab === 'footing' && (
              <>
                <Input label="ទទឹងជើងតាង (W)" value={footing.w} onChange={v => setFooting({...footing, w: v})} unit="m" />
                <Input label="បណ្តោយ (L)" value={footing.l} onChange={v => setFooting({...footing, l: v})} unit="m" />
                <Input label="កម្រាស់ (D)" value={footing.d} onChange={v => setFooting({...footing, d: v})} unit="m" />
                <Input label="ចំនួន" value={footing.qty} onChange={v => setFooting({...footing, qty: v})} unit="កន្លែង" />
                <Input label="ទំហំដែក" value={footing.rebarSize} onChange={v => setFooting({...footing, rebarSize: v})} unit="mm" />
                <Input label="ចន្លោះដែក" value={footing.spacing} onChange={v => setFooting({...footing, spacing: v})} unit="mm" />
                <div className="col-span-2 mt-2 bg-slate-50 p-4 rounded-[1.5rem]">
                  <label className="text-[10px] font-black text-slate-400 block mb-3 text-center uppercase">ចំនួនស្រទាប់ដែក</label>
                  <div className="flex gap-3">
                    {[1, 2].map(l => (
                      <button key={l} onClick={() => setFooting({...footing, layers: l})} className={`flex-1 py-3 rounded-xl text-xs font-black transition-all ${footing.layers === l ? 'bg-slate-800 text-white' : 'bg-white text-slate-400 border border-slate-200'}`}>
                        {l} ស្រទាប់
                      </button>
                    ))}
                  </div>
                </div>
              </>
            )}
            {activeTab === 'slab' && (
              <>
                <Input label="ទទឹងប្លង់សេ (W)" value={slab.w} onChange={v => setSlab({...slab, w: v})} unit="m" />
                <Input label="បណ្តោយ (L)" value={slab.l} onChange={v => setSlab({...slab, l: v})} unit="m" />
                <Input label="កម្រាស់ (D)" value={slab.d} onChange={v => setSlab({...slab, d: v})} unit="m" />
                <Input label="ចំនួន" value={slab.qty} onChange={v => setSlab({...slab, qty: v})} unit="ផ្ទាំង" />
                <Input label="ទំហំដែក" value={slab.rebarSize} onChange={v => setSlab({...slab, rebarSize: v})} unit="mm" />
                <Input label="ចន្លោះដែក" value={slab.spacing} onChange={v => setSlab({...slab, spacing: v})} unit="mm" />
                <div className="col-span-2 mt-2 bg-slate-50 p-4 rounded-[1.5rem]">
                  <label className="text-[10px] font-black text-slate-400 block mb-3 text-center uppercase">ចំនួនស្រទាប់ដែក</label>
                  <div className="flex gap-3">
                    {[1, 2].map(l => (
                      <button key={l} onClick={() => setSlab({...slab, layers: l})} className={`flex-1 py-3 rounded-xl text-xs font-black transition-all ${slab.layers === l ? 'bg-slate-800 text-white' : 'bg-white text-slate-400 border border-slate-200'}`}>
                        {l} ស្រទាប់
                      </button>
                    ))}
                  </div>
                </div>
              </>
            )}
            {activeTab === 'column' && (
              <>
                <Input label="ទទឹងសសរ (W)" value={column.w} onChange={v => setColumn({...column, w: v})} unit="m" />
                <Input label="បណ្តោយ (L)" value={column.l} onChange={v => setColumn({...column, l: v})} unit="m" />
                <Input label="កម្ពស់សសរ (H)" value={column.h} onChange={v => setColumn({...column, h: v})} unit="m" />
                <Input label="ចំនួនសរុប" value={column.qty} onChange={v => setColumn({...column, qty: v})} unit="ដើម" />
                <Input label="ដែកមេ" value={column.mainRebar} onChange={v => setColumn({...column, mainRebar: v})} unit="សសៃ" />
                <Input label="ទំហំដែកមេ" value={column.mainSize} onChange={v => setColumn({...column, mainSize: v})} unit="mm" />
                <Input label="ទំហំដែកក្លេ" value={column.stirrupSize} onChange={v => setColumn({...column, stirrupSize: v})} unit="mm" />
                <Input label="ចន្លោះក្លេ" value={column.stirrupSpacing} onChange={v => setColumn({...column, stirrupSpacing: v})} unit="mm" />
              </>
            )}
            {activeTab === 'beam' && (
              <>
                <Input label="ទទឹងធ្នឹម (W)" value={beam.w} onChange={v => setBeam({...beam, w: v})} unit="m" />
                <Input label="កម្ពស់ធ្នឹម (H)" value={beam.h} onChange={(v) => setBeam({...beam, h: v})} unit="m" />
                <Input label="ប្រវែងធ្នឹម (L)" value={beam.l} onChange={(v) => setBeam({...beam, l: v})} unit="m" />
                <Input label="ចំនួនសរុប" value={beam.qty} onChange={(v) => setBeam({...beam, qty: v})} unit="ភ្នែង" />
                <Input label="ដែកលើ" value={beam.mainTop} onChange={(v) => setBeam({...beam, mainTop: v})} unit="សសៃ" />
                <Input label="ដែកក្រោម" value={beam.mainBottom} onChange={(v) => setBeam({...beam, mainBottom: v})} unit="សសៃ" />
                <Input label="ទំហំដែកមេ" value={beam.mainSize} onChange={(v) => setBeam({...beam, mainSize: v})} unit="mm" />
                <Input label="ចន្លោះក្លេ" value={beam.stirrupSpacing} onChange={(v) => setBeam({...beam, stirrupSpacing: v})} unit="mm" />
              </>
            )}
          </div>
        </div>

        {/* Results Panel */}
        <div className="bg-slate-900 px-8 pt-10 pb-12 rounded-t-[4rem] text-white">
          <div className="grid grid-cols-2 gap-6 mb-8">
            <div>
              <p className="text-[10px] font-black text-blue-400 uppercase tracking-widest mb-1">មាឌបេតុងសរុប</p>
              <div className="text-4xl font-black">{volume.toFixed(2)} <span className="text-sm font-medium opacity-40">m³</span></div>
            </div>
            <div className="text-right">
              <p className="text-[10px] font-black text-yellow-400 uppercase tracking-widest mb-1">ដែកសរុប</p>
              <div className="text-3xl font-black">{steel} <span className="text-sm font-medium opacity-40">kg</span></div>
            </div>
          </div>

          <div className="space-y-3 mb-8">
            <div className="flex justify-between items-center bg-white/5 p-4 rounded-2xl border border-white/5">
              <span className="text-xs text-slate-400 font-bold uppercase tracking-wider">ស៊ីម៉ងត៍ (50kg)</span>
              <span className="font-black text-white">{(volume * mix.cement / 50).toFixed(1)} <span className="text-[10px] font-medium opacity-40 uppercase ml-1">បាវ</span></span>
            </div>
            
            <div className="flex justify-between items-center bg-blue-500/10 p-4 rounded-2xl border border-blue-500/20">
              <div className="flex items-center gap-2">
                <Droplets size={16} className="text-blue-400" />
                <span className="text-xs text-blue-400 font-bold uppercase tracking-wider">ទឹកស្អាតសរុប</span>
              </div>
              <span className="font-black text-white">{(volume * mix.water).toFixed(0)} <span className="text-[10px] font-medium opacity-40 uppercase ml-1">លីត្រ</span></span>
            </div>

            <div className="grid grid-cols-2 gap-3">
              <div className="bg-white/5 p-4 rounded-2xl border border-white/5">
                <span className="text-[10px] text-slate-500 font-bold uppercase block mb-1 text-center">ខ្សាច់ (Sand)</span>
                <span className="text-sm font-black text-white block text-center">{(volume * mix.sand).toFixed(2)} <span className="text-[10px] font-normal opacity-40 uppercase">m³</span></span>
              </div>
              <div className="bg-white/5 p-4 rounded-2xl border border-white/5">
                <span className="text-[10px] text-slate-500 font-bold uppercase block mb-1 text-center">ថ្ម (Stone)</span>
                <span className="text-sm font-black text-white block text-center">{(volume * mix.stone).toFixed(2)} <span className="text-[10px] font-normal opacity-40 uppercase">m³</span></span>
              </div>
            </div>
          </div>

          {/* Footer Branding */}
          <div className="pt-6 border-t border-white/5 flex flex-col items-center gap-2">
            <div className="flex items-center gap-2 text-[10px] font-black text-blue-400 uppercase tracking-[0.3em]">
              <Copyright size={12} /> Nem Linda 2026
            </div>
            <p className="text-[9px] text-slate-500 text-center leading-relaxed italic opacity-60">
              Professional Engineering Estimation Tool.
            </p>
          </div>
        </div>
      </div>
      
      <div className="mt-8 max-w-md w-full px-6 flex items-start gap-3 bg-white/50 p-4 rounded-2xl backdrop-blur-sm border border-white">
        <Info size={20} className="text-blue-500 shrink-0 mt-0.5" />
        <p className="text-[10px] text-slate-500 leading-relaxed font-medium">
          <strong>ចំណាំ៖</strong> ការគណនានេះផ្អែកលើស្តង់ដារល្បាយបេតុង និងរូបមន្តដែក $D^2/162$។ លទ្ធផលអាចមានការប្រែប្រួលតិចតួចតាមការអនុវត្តជាក់ស្តែងនៅការដ្ឋាន។
        </p>
      </div>
    </div>
  );
};

export default App;
