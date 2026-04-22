/**
 * @license
 * SPDX-License-Identifier: Apache-2.0
 */

import { useState, useRef, ChangeEvent } from 'react';
import { motion, AnimatePresence } from 'motion/react';
import { 
  Upload, 
  FileAudio, 
  Loader2, 
  Download, 
  CheckCircle, 
  AlertCircle, 
  FileText,
  Volume2,
  Globe2,
  Sparkles
} from 'lucide-react';
import ReactMarkdown from 'react-markdown';
import { transcribeAudio, generateReport } from './lib/gemini';

// Helper to convert File to base64 with progress tracking
const fileToBase64 = (file: File, onProgress: (percent: number) => void): Promise<string> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    
    reader.onprogress = (event) => {
      if (event.lengthComputable) {
        const percent = Math.round((event.loaded / event.total) * 100);
        onProgress(percent);
      }
    };

    reader.onload = () => {
      const result = reader.result?.toString();
      if (result) {
        const base64String = result.split(',')[1];
        resolve(base64String);
      } else {
        reject(new Error("Failed to convert file to base64"));
      }
    };
    reader.onerror = (error) => reject(error);
    reader.readAsDataURL(file);
  });
};

export default function App() {
  const [file, setFile] = useState<File | null>(null);
  const [transcription, setTranscription] = useState<string>('');
  const [report, setReport] = useState<string>('');
  const [activeTab, setActiveTab] = useState<'transp' | 'report'>('transp');
  const [isLoading, setIsLoading] = useState(false);
  const [isGeneratingReport, setIsGeneratingReport] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [progress, setProgress] = useState<string>('');
  const [uploadPercent, setUploadPercent] = useState(0);
  const [transcribePercent, setTranscribePercent] = useState(0);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const handleFileChange = (e: ChangeEvent<HTMLInputElement>) => {
    const selectedFile = e.target.files?.[0];
    if (selectedFile) {
      setFile(selectedFile);
      setError(null);
      setTranscription('');
      setReport('');
      setActiveTab('transp');
      setUploadPercent(0);
      setTranscribePercent(0);
    }
  };

  const handleTranscribe = async () => {
    if (!file) return;

    setIsLoading(true);
    setError(null);
    setTranscribePercent(0);
    setProgress('Chargement du fichier...');
    
    try {
      const base64 = await fileToBase64(file, (p) => setUploadPercent(p));
      
      setProgress('Analyse IA (ceci peut prendre 30-90s)...');
      
      // Simulate transcription progress since we can't get real % from one-shot API
      const interval = setInterval(() => {
        setTranscribePercent(prev => {
          if (prev >= 95) {
            clearInterval(interval);
            return 95;
          }
          return prev + (prev < 50 ? 5 : 2);
        });
      }, 1500);

      const result = await transcribeAudio(base64, file.type);
      
      clearInterval(interval);
      setTranscribePercent(100);
      setTranscription(result);
    } catch (err) {
      console.error(err);
      setError("Une erreur est survenue lors de la transcription. Assurez-vous que la clé API est valide.");
    } finally {
      setIsLoading(false);
      setProgress('');
    }
  };

  const handleGenerateReport = async () => {
    if (!transcription) return;

    setIsGeneratingReport(true);
    setError(null);
    setActiveTab('report');
    
    try {
      const result = await generateReport(transcription);
      setReport(result);
    } catch (err) {
      console.error(err);
      setError("Erreur lors de la génération du rapport.");
    } finally {
      setIsGeneratingReport(false);
    }
  };

  const downloadFile = (content: string, type: string) => {
    const element = document.createElement("a");
    const fileBlob = new Blob([content], {type: 'text/plain'});
    element.href = URL.createObjectURL(fileBlob);
    element.download = `${type}_${file?.name || 'result'}.txt`;
    document.body.appendChild(element);
    element.click();
    document.body.removeChild(element);
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900 font-sans p-4 md:p-8 selection:bg-indigo-100 selection:text-indigo-900">
      <div className="max-w-7xl mx-auto grid grid-cols-1 md:grid-cols-12 md:grid-rows-6 gap-4">
        
        {/* Header Card */}
        <div className="md:col-span-8 md:row-span-1 bento-card flex items-center justify-between">
          <div className="flex items-center gap-4">
            <div className="w-10 h-10 bg-indigo-600 rounded-xl flex items-center justify-center text-white shadow-lg shadow-indigo-100">
              <Globe2 className="w-6 h-6" />
            </div>
            <div>
              <h1 className="font-bold text-lg tracking-tight">Transcribe Maroc <span className="text-indigo-600">v2.1</span></h1>
              <p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">Transcription Intégrale • Sans Limite</p>
            </div>
          </div>
          <div className="hidden sm:flex gap-2">
            <span className={`px-3 py-1 text-[10px] font-bold uppercase tracking-wider rounded-full border ${isLoading ? 'bg-amber-50 text-amber-700 border-amber-100' : transcription ? 'bg-emerald-50 text-emerald-700 border-emerald-100' : 'bg-slate-100 text-slate-500 border-slate-200'}`}>
              {isLoading ? 'Traitement en cours' : transcription ? 'Terminé' : 'En attente'}
            </span>
          </div>
        </div>

        {/* Controls Card */}
        <div className="md:col-span-4 md:row-span-2 bento-card flex flex-col justify-between">
          <div className="space-y-4">
            <h2 className="text-[10px] font-bold text-slate-400 uppercase tracking-widest">Import Audio / Vidéo</h2>
            
            <div 
              onClick={() => fileInputRef.current?.click()}
              className={`
                group relative flex flex-col items-center justify-center p-6 border-2 border-dashed rounded-xl cursor-pointer transition-all
                ${file ? 'border-indigo-600 bg-indigo-50/50' : 'border-slate-200 hover:border-indigo-400 hover:bg-slate-50'}
              `}
            >
              <input 
                type="file" 
                ref={fileInputRef}
                onChange={handleFileChange}
                className="hidden" 
                accept="audio/*,video/*"
              />
              
              {file ? (
                <div className="flex flex-col items-center gap-2 text-center">
                  <div className="p-2 bg-indigo-600 rounded-lg">
                    <FileAudio className="w-5 h-5 text-white" />
                  </div>
                  <div className="space-y-0.5">
                    <p className="text-xs font-semibold truncate max-w-[150px]">{file.name}</p>
                    <p className="text-[10px] text-slate-400">{(file.size / (1024 * 1024)).toFixed(2)} MB</p>
                  </div>
                </div>
              ) : (
                <div className="flex flex-col items-center gap-3 text-center">
                  <Upload className="w-5 h-5 text-slate-300 group-hover:text-indigo-500 transition-colors" />
                  <p className="text-[11px] text-slate-400">
                    Déposez ou <span className="text-indigo-600 font-semibold">parcourez</span>
                  </p>
                </div>
              )}
            </div>
          </div>

          <button
            disabled={!file || isLoading}
            onClick={handleTranscribe}
            className={`
              w-full py-3 mt-4 rounded-xl font-bold uppercase tracking-widest text-[11px] transition-all flex items-center justify-center gap-2 shadow-sm
              ${!file || isLoading 
                ? 'bg-slate-100 text-slate-300 cursor-not-allowed border border-slate-200' 
                : 'bg-indigo-600 text-white hover:bg-indigo-700 active:transform active:scale-95 shadow-indigo-100'}
            `}
          >
            {isLoading ? (
              <>
                <Loader2 className="w-3 h-3 animate-spin" />
                Processing...
              </>
            ) : (
              <>
                <Volume2 className="w-3 h-3" />
                Transcrire
              </>
            )}
          </button>
        </div>

        {/* Transcription/Report Card */}
        <div className="md:col-span-8 md:row-span-5 bento-card overflow-hidden flex flex-col p-0!">
          <div className="bg-slate-50/50 px-6 py-3 border-b border-slate-100 flex justify-between items-center shrink-0">
            <div className="flex gap-4 text-[10px] font-bold uppercase tracking-tighter">
              <button 
                onClick={() => setActiveTab('transp')}
                className={`transition-colors ${activeTab === 'transp' ? 'text-indigo-600' : 'text-slate-400 hover:text-slate-600'}`}
              >
                Transcription
              </button>
              <button 
                disabled={!transcription}
                onClick={() => setActiveTab('report')}
                className={`transition-colors ${!transcription ? 'opacity-30 cursor-not-allowed' : activeTab === 'report' ? 'text-indigo-600' : 'text-slate-400 hover:text-slate-600'}`}
              >
                Rapport Rédigé
              </button>
            </div>
            <div className="flex gap-1.5">
              <div className="h-2 w-2 rounded-full bg-slate-200"></div>
              <div className="h-2 w-2 rounded-full bg-slate-200"></div>
              <div className="h-2 w-2 rounded-full bg-slate-200"></div>
            </div>
          </div>

          <div className="flex-1 overflow-y-auto p-8 relative scrollbar-thin scrollbar-thumb-slate-200 scrollbar-track-transparent">
            {error && (
              <motion.div 
                initial={{ opacity: 0, scale: 0.95 }}
                animate={{ opacity: 1, scale: 1 }}
                className="flex gap-2 p-4 mb-6 bg-red-50 border border-red-100 rounded-xl text-red-600 text-xs font-medium"
              >
                <AlertCircle className="w-4 h-4 shrink-0" />
                <p>{error}</p>
              </motion.div>
            )}

            <AnimatePresence mode="wait">
              {isLoading || isGeneratingReport ? (
                <motion.div 
                  key="loading"
                  initial={{ opacity: 0 }}
                  animate={{ opacity: 1 }}
                  exit={{ opacity: 0 }}
                  className="absolute inset-0 flex flex-col items-center justify-center p-12 text-center"
                >
                  <div className="w-16 h-16 relative mb-8">
                    <Loader2 className="w-16 h-16 text-indigo-600 animate-spin" />
                  </div>
                  
                  <div className="w-full max-w-sm space-y-8">
                    <div className="space-y-2">
                       <div className="flex justify-between text-[10px] font-bold uppercase tracking-widest text-slate-400">
                         <span>Chargement Fichier</span>
                         <span>{uploadPercent}%</span>
                       </div>
                       <div className="h-1.5 w-full bg-slate-100 rounded-full overflow-hidden">
                         <motion.div 
                          className="h-full bg-emerald-500"
                          initial={{ width: 0 }}
                          animate={{ width: `${uploadPercent}%` }}
                         />
                       </div>
                    </div>

                    {!isGeneratingReport && uploadPercent === 100 && (
                      <motion.div 
                        initial={{ opacity: 0, y: 10 }}
                        animate={{ opacity: 1, y: 0 }}
                        className="space-y-2"
                      >
                         <div className="flex justify-between text-[10px] font-bold uppercase tracking-widest text-slate-400">
                           <span>Analyse IA en cours</span>
                           <span>{transcribePercent}%</span>
                         </div>
                         <div className="h-1.5 w-full bg-slate-100 rounded-full overflow-hidden">
                           <motion.div 
                            className="h-full bg-indigo-600"
                            initial={{ width: 0 }}
                            animate={{ width: `${transcribePercent}%` }}
                           />
                         </div>
                         <p className="text-[10px] text-slate-400 italic">"Traitement de la Darija et du Français..."</p>
                      </motion.div>
                    )}

                    <div className="space-y-1">
                      <p className="text-sm font-bold text-slate-900 uppercase tracking-widest">
                        {isGeneratingReport ? 'Rédaction du rapport...' : progress}
                      </p>
                    </div>
                  </div>
                </motion.div>
              ) : activeTab === 'report' && report ? (
                <motion.div 
                  key="report"
                  initial={{ opacity: 0, y: 10 }}
                  animate={{ opacity: 1, y: 0 }}
                  className="markdown-body prose prose-slate max-w-none 
                    prose-p:text-slate-700 prose-p:leading-relaxed prose-p:text-sm
                    prose-strong:text-indigo-600 prose-strong:font-bold
                    prose-headings:text-slate-900 prose-headings:font-display"
                >
                  <ReactMarkdown>{report}</ReactMarkdown>
                </motion.div>
              ) : activeTab === 'transp' && transcription ? (
                <motion.div 
                  key="transcription"
                  initial={{ opacity: 0, y: 10 }}
                  animate={{ opacity: 1, y: 0 }}
                  className="markdown-body prose prose-slate max-w-none 
                    prose-p:text-slate-700 prose-p:leading-relaxed prose-p:text-sm
                    prose-strong:text-indigo-600 prose-strong:font-bold
                    prose-headings:text-slate-900 prose-headings:font-display"
                >
                  <ReactMarkdown>{transcription}</ReactMarkdown>
                </motion.div>
              ) : (
                <div className="absolute inset-0 flex flex-col items-center justify-center text-center p-12 opacity-30">
                  <Volume2 className="w-16 h-16 mb-4 text-slate-400" />
                  <p className="text-sm font-medium text-slate-400 tracking-tight">En attente d'analyse...</p>
                </div>
              )}
            </AnimatePresence>
          </div>

          <div className="p-4 bg-slate-50 border-t border-slate-200 flex gap-2 items-center shrink-0">
            <div className="flex-1 bg-white rounded-lg h-10 border border-slate-200 flex items-center px-4 gap-4">
              <div className={`w-2 h-2 rounded-full ${isLoading || isGeneratingReport ? 'bg-amber-500 animate-pulse' : (transcription || report) ? 'bg-emerald-500' : 'bg-slate-200'}`}></div>
              <div className="flex gap-0.5 items-end h-3">
                {[...Array(12)].map((_, i) => (
                  <div key={i} className={`w-1 rounded-full ${isLoading || isGeneratingReport ? 'bg-indigo-300' : 'bg-slate-200'}`} style={{ height: `${Math.random() * 100}%` }}></div>
                ))}
              </div>
              <div className="text-[10px] font-bold text-slate-400 uppercase tracking-tighter">Flux Actif</div>
            </div>
            
            {transcription && (
              <div className="flex gap-2">
                <button 
                  onClick={() => downloadFile(activeTab === 'report' ? report : transcription, activeTab === 'report' ? 'rapport' : 'transcription')}
                  className="p-2.5 bg-white border border-slate-200 rounded-lg hover:bg-slate-50 transition-colors"
                  title="Télécharger"
                >
                  <Download className="w-4 h-4 text-slate-600" />
                </button>
              </div>
            )}
          </div>
        </div>

        {/* Action Card: Generate Report */}
        <button 
          disabled={!transcription || isLoading || isGeneratingReport}
          onClick={handleGenerateReport}
          className={`
            md:col-span-4 md:row-span-2 bento-card flex flex-col items-center justify-center gap-4 group transition-all
            ${!transcription || isLoading || isGeneratingReport ? 'opacity-30 cursor-not-allowed' : 'hover:border-indigo-400 hover:shadow-md cursor-pointer'}
          `}
        >
          <div className="p-4 bg-indigo-50 rounded-2xl group-hover:bg-indigo-100 transition-colors">
            <Sparkles className="w-8 h-8 text-indigo-600" />
          </div>
          <div className="text-center">
            <p className="text-[10px] font-bold text-slate-400 uppercase tracking-[0.2em] mb-1">IA Writer</p>
            <p className="text-sm font-bold text-slate-900 uppercase">Générer un Rapport</p>
            <p className="text-[10px] text-slate-400 mt-2 italic">Format professionnel structuré</p>
          </div>
        </button>

        {/* Info Card */}
        <div className="md:col-span-4 md:row-span-2 bento-card">
          <h2 className="text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-4">Statut Système</h2>
          <div className="space-y-4">
            <div className="flex items-center justify-between text-[11px] font-medium text-slate-600">
              <span className="flex items-center gap-2"><div className="w-1.5 h-1.5 rounded-full bg-emerald-500"></div>Mode Sans Limite</span>
              <span className="text-[9px] px-2 py-0.5 bg-emerald-50 text-emerald-600 rounded">Actif</span>
            </div>
            <div className="flex items-center justify-between text-[11px] font-medium text-slate-600">
              <span className="flex items-center gap-2"><div className="w-1.5 h-1.5 rounded-full bg-indigo-500"></div>Bilingue (FR/DA)</span>
              <span className="text-[9px] px-2 py-0.5 bg-indigo-50 text-indigo-600 rounded">Optimisé</span>
            </div>
            <div className="pt-4 border-t border-slate-100">
              <p className="text-[9px] text-slate-400 leading-relaxed italic">
                Note: Les fichiers volumineux sont acceptés sans limite de taille artificielle. 
                Le temps de traitement dépendra de la durée de l'audio.
              </p>
            </div>
          </div>
        </div>

      </div>
    </div>
  );
}
