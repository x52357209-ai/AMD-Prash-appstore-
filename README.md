import React, { useState, useEffect } from 'react';
import { 
  TrendingUp, 
  Video, 
  Image as ImageIcon, 
  Upload, 
  Zap, 
  Copy, 
  Check, 
  Layout, 
  Settings, 
  Youtube, 
  Loader2, 
  RefreshCw,
  Sparkles,
  Save
} from 'lucide-react';

export default function App() {
  const [activeTab, setActiveTab] = useState('trends');
  const [apiKey, setApiKey] = useState(''); // In a real app, this would be handled securely
  const [generatedContent, setGeneratedContent] = useState(null);
  const [loading, setLoading] = useState(false);
  const [projects, setProjects] = useState([]);
  const [notification, setNotification] = useState(null);

  // Simulation of trending topics
  const trendingTopics = [
    { id: 1, title: "AI Productivity Hacks", views: "2.4M", category: "Tech" },
    { id: 2, title: "Unbelievable Space Facts", views: "1.8M", category: "Science" },
    { id: 3, title: "5 Minute Healthy Meals", views: "3.1M", category: "Food" },
    { id: 4, title: "Psychology Tricks", views: "5.2M", category: "Education" },
    { id: 5, title: "Hidden iPhone Features", views: "900K", category: "Tech" },
    { id: 6, title: "Motivation for 2025", views: "1.2M", category: "Lifestyle" },
  ];

  const showNotification = (message, type = 'success') => {
    setNotification({ message, type });
    setTimeout(() => setNotification(null), 3000);
  };

  const handleGenerateScript = async (topic) => {
    setLoading(true);
    setActiveTab('studio');
    
    // Determine which Gemini model to use based on environment
    // Note: In this environment, we use a proxy that injects the key, 
    // but for the code to be 'correct' we structure the payload for the proxy.
    
    const prompt = `Write a viral YouTube Shorts script about "${topic}". 
    Include:
    1. A Hook (0-3 seconds)
    2. Main Value (3-45 seconds)
    3. Call to Action (45-60 seconds)
    4. 3 Viral Title Options
    5. A short description with hashtags.
    
    Format the response as JSON.`;

    try {
      // Simulation of API call delay for realism if real API fails or for UX
      // Real implementation would fetch from: https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent
      
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: prompt }] }],
          generationConfig: { responseMimeType: "application/json" }
        })
      });

      const data = await response.json();
      
      let parsedContent;
      if (data.candidates && data.candidates[0].content) {
         parsedContent = JSON.parse(data.candidates[0].content.parts[0].text);
      } else {
         // Fallback mock data if API fails or key is missing
         parsedContent = {
           script: {
             hook: "Stop scrolling! You won't believe this...",
             body: `Here is the mind-blowing fact about ${topic}. Most people think X, but actually Y.`,
             cta: "Subscribe for more daily secrets!"
           },
           titles: [`Why ${topic} is Changing Everything`, `The Secret of ${topic}`, `Don't Ignore ${topic}`],
           description: `#shorts #${topic.replace(/\s/g, '')} #viral`
         };
         if(apiKey) showNotification("Using mock data (API check failed)", "error");
      }

      setGeneratedContent({
        topic,
        ...parsedContent,
        thumbnail: null
      });
      showNotification("Script generated successfully!");

    } catch (error) {
      console.error(error);
      showNotification("Error generating script. Using simulation.", "error");
      // Fallback
      setGeneratedContent({
        topic,
        script: { hook: "Did you know?", body: `Amazing facts about ${topic}...`, cta: "Like for part 2!" },
        titles: [`Top 3 Facts About ${topic}`, `${topic} Explained`],
        description: `#shorts #${topic.replace(/\s/g, '')}`
      });
    } finally {
      setLoading(false);
    }
  };

  const handleGenerateThumbnail = async () => {
    if (!generatedContent) return;
    setLoading(true);

    const prompt = `A high quality, vibrant YouTube thumbnail for a video about ${generatedContent.topic}. High contrast, 4k, hyper-realistic, catchy, bright colors, no text.`;

    try {
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          instances: [{ prompt: prompt }],
          parameters: { sampleCount: 1 }
        })
      });

      const data = await response.json();
      
      if (data.predictions && data.predictions[0]) {
        const imageUrl = `data:image/png;base64,${data.predictions[0].bytesBase64Encoded}`;
        setGeneratedContent(prev => ({ ...prev, thumbnail: imageUrl }));
        showNotification("Thumbnail generated!");
      } else {
        throw new Error("No image returned");
      }
    } catch (error) {
      // Fallback placeholder
      setGeneratedContent(prev => ({ 
        ...prev, 
        thumbnail: "https://placehold.co/600x900/1a1a1a/FFF?text=Thumbnail+Generation+Failed" 
      }));
      showNotification("Failed to generate image (Check API Key)", "error");
    } finally {
      setLoading(false);
    }
  };

  const handleUploadSimulation = () => {
    setLoading(true);
    setTimeout(() => {
      setProjects([generatedContent, ...projects]);
      setGeneratedContent(null);
      setLoading(false);
      setActiveTab('uploads');
      showNotification("Video scheduled for upload!");
    }, 2000);
  };

  // --- Components ---

  const SidebarItem = ({ id, icon: Icon, label }) => (
    <button
      onClick={() => setActiveTab(id)}
      className={`w-full flex items-center gap-3 px-4 py-3 rounded-xl transition-all duration-200 ${
        activeTab === id 
          ? 'bg-red-600 text-white shadow-lg shadow-red-900/20' 
          : 'text-slate-400 hover:bg-slate-800 hover:text-white'
      }`}
    >
      <Icon size={20} />
      <span className="font-medium">{label}</span>
    </button>
  );

  const TrendsView = () => (
    <div className="space-y-6 animate-in fade-in duration-500">
      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold text-white flex items-center gap-2">
          <TrendingUp className="text-red-500" /> Trending Now
        </h2>
        <span className="text-sm text-slate-500">Updated 5m ago</span>
      </div>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {trendingTopics.map((topic) => (
          <div key={topic.id} className="bg-slate-800/50 border border-slate-700 p-5 rounded-2xl hover:border-red-500/50 transition-all group">
            <div className="flex justify-between items-start mb-3">
              <span className="bg-slate-700 text-xs text-slate-300 px-2 py-1 rounded-full">{topic.category}</span>
              <span className="flex items-center gap-1 text-xs text-green-400 font-mono">
                <TrendingUp size={12} /> {topic.views}
              </span>
            </div>
            <h3 className="text-lg font-semibold text-white mb-4 group-hover:text-red-400 transition-colors">{topic.title}</h3>
            <button 
              onClick={() => handleGenerateScript(topic.title)}
              className="w-full py-2 bg-slate-700 hover:bg-red-600 text-white rounded-lg text-sm font-medium transition-colors flex items-center justify-center gap-2"
            >
              <Zap size={16} /> Analyze & Create
            </button>
          </div>
        ))}
      </div>
    </div>
  );

  const StudioView = () => (
    <div className="max-w-4xl mx-auto space-y-6 animate-in slide-in-from-bottom-4 duration-500">
      {!generatedContent ? (
        <div className="text-center py-20">
          <div className="w-20 h-20 bg-slate-800 rounded-full flex items-center justify-center mx-auto mb-6">
            <Sparkles className="text-slate-500" size={40} />
          </div>
          <h2 className="text-2xl font-bold text-white mb-2">Ready to Create?</h2>
          <p className="text-slate-400 mb-6">Select a trending topic from the dashboard to start generating content.</p>
          <button 
            onClick={() => setActiveTab('trends')}
            className="px-6 py-3 bg-red-600 text-white rounded-xl font-medium hover:bg-red-700 transition-colors"
          >
            Go to Trends
          </button>
        </div>
      ) : (
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
          {/* Left Column: Script & Meta */}
          <div className="space-y-6">
            <div className="bg-slate-800 border border-slate-700 rounded-2xl p-6">
              <div className="flex items-center gap-2 mb-4 text-purple-400">
                <Video size={20} />
                <h3 className="font-bold uppercase tracking-wider text-sm">AI Script</h3>
              </div>
              
              <div className="space-y-4">
                <div className="bg-slate-900/50 p-4 rounded-xl border border-slate-700/50">
                  <span className="text-xs font-bold text-slate-500 uppercase">Hook (0-3s)</span>
                  <p className="text-white mt-1">{generatedContent.script.hook}</p>
                </div>
                <div className="bg-slate-900/50 p-4 rounded-xl border border-slate-700/50">
                  <span className="text-xs font-bold text-slate-500 uppercase">Core Content</span>
                  <p className="text-slate-300 mt-1">{generatedContent.script.body}</p>
                </div>
                <div className="bg-slate-900/50 p-4 rounded-xl border border-slate-700/50">
                  <span className="text-xs font-bold text-slate-500 uppercase">Call to Action</span>
                  <p className="text-red-400 mt-1">{generatedContent.script.cta}</p>
                </div>
              </div>
            </div>

            <div className="bg-slate-800 border border-slate-700 rounded-2xl p-6">
               <h3 className="font-bold text-slate-200 mb-4">Metadata</h3>
               <div className="space-y-3">
                 {generatedContent.titles.map((title, i) => (
                   <div key={i} className="flex items-center gap-3 p-3 bg-slate-900/50 rounded-lg group cursor-pointer hover:bg-slate-700 transition-colors">
                     <div className="w-6 h-6 rounded-full bg-slate-700 text-slate-400 flex items-center justify-center text-xs group-hover:bg-red-500 group-hover:text-white">
                       {i + 1}
                     </div>
                     <p className="text-sm text-slate-300 flex-1">{title}</p>
                     <Copy size={14} className="text-slate-600 group-hover:text-white" />
                   </div>
                 ))}
               </div>
            </div>
          </div>

          {/* Right Column: Visuals & Actions */}
          <div className="space-y-6">
            <div className="bg-slate-800 border border-slate-700 rounded-2xl p-6">
              <div className="flex items-center justify-between mb-4">
                <div className="flex items-center gap-2 text-pink-400">
                  <ImageIcon size={20} />
                  <h3 className="font-bold uppercase tracking-wider text-sm">Thumbnail</h3>
                </div>
                <button 
                  onClick={handleGenerateThumbnail}
                  disabled={loading}
                  className="text-xs bg-slate-700 hover:bg-slate-600 text-white px-3 py-1 rounded-full transition-colors flex items-center gap-1"
                >
                  <RefreshCw size={12} className={loading ? "animate-spin" : ""} /> Regenerate
                </button>
              </div>

              <div className="aspect-[9/16] bg-slate-900 rounded-xl overflow-hidden border-2 border-dashed border-slate-700 flex items-center justify-center relative group">
                {generatedContent.thumbnail ? (
                  <img src={generatedContent.thumbnail} alt="Generated Thumbnail" className="w-full h-full object-cover" />
                ) : (
                  <div className="text-center p-6">
                     <div className="w-16 h-16 bg-slate-800 rounded-full flex items-center justify-center mx-auto mb-4 group-hover:scale-110 transition-transform">
                       <ImageIcon className="text-slate-600" size={32} />
                     </div>
                     <p className="text-slate-500 text-sm">Click regenerate to create an AI thumbnail</p>
                  </div>
                )}
                {loading && (
                  <div className="absolute inset-0 bg-black/60 flex items-center justify-center backdrop-blur-sm">
                    <Loader2 className="animate-spin text-white" size={32} />
                  </div>
                )}
              </div>
            </div>

            <button 
              onClick={handleUploadSimulation}
              disabled={loading || !generatedContent.thumbnail}
              className={`w-full py-4 rounded-xl font-bold text-lg flex items-center justify-center gap-2 transition-all ${
                loading || !generatedContent.thumbnail
                ? 'bg-slate-700 text-slate-500 cursor-not-allowed'
                : 'bg-red-600 hover:bg-red-700 text-white shadow-lg shadow-red-900/30 hover:shadow-red-900/50 hover:-translate-y-1'
              }`}
            >
              {loading ? (
                <>Processing...</>
              ) : (
                <>
                  <Upload size={24} /> Upload to Channel
                </>
              )}
            </button>
            <p className="text-center text-xs text-slate-500">
              *Uploads are simulated in this demo environment.
            </p>
          </div>
        </div>
      )}
    </div>
  );

  const UploadsView = () => (
    <div className="space-y-6 animate-in fade-in duration-500">
       <h2 className="text-2xl font-bold text-white flex items-center gap-2">
          <Youtube className="text-red-500" /> Channel Content
        </h2>
        
        {projects.length === 0 ? (
          <div className="text-center py-20 bg-slate-800/30 rounded-2xl border border-dashed border-slate-700">
            <p className="text-slate-400">No videos scheduled yet.</p>
          </div>
        ) : (
          <div className="bg-slate-800 border border-slate-700 rounded-2xl overflow-hidden">
            <table className="w-full text-left">
              <thead className="bg-slate-900 text-slate-400 text-xs uppercase">
                <tr>
                  <th className="px-6 py-4 font-semibold">Video</th>
                  <th className="px-6 py-4 font-semibold">Status</th>
                  <th className="px-6 py-4 font-semibold">Date</th>
                  <th className="px-6 py-4 font-semibold">Views (Est)</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-700">
                {projects.map((p, i) => (
                  <tr key={i} className="text-sm text-slate-300 hover:bg-slate-700/50 transition-colors">
                    <td className="px-6 py-4">
                      <div className="flex items-center gap-4">
                        <div className="w-10 h-16 bg-slate-900 rounded overflow-hidden flex-shrink-0">
                          {p.thumbnail && <img src={p.thumbnail} className="w-full h-full object-cover" />}
                        </div>
                        <div>
                          <p className="font-medium text-white line-clamp-1">{p.titles[0]}</p>
                          <p className="text-xs text-slate-500 line-clamp-1">{p.script.hook}</p>
                        </div>
                      </div>
                    </td>
                    <td className="px-6 py-4">
                      <span className="bg-yellow-500/10 text-yellow-500 border border-yellow-500/20 px-2 py-1 rounded-full text-xs font-medium">
                        Scheduled
                      </span>
                    </td>
                    <td className="px-6 py-4 text-slate-500">
                      Just now
                    </td>
                    <td className="px-6 py-4 font-mono text-slate-500">
                      --
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        )}
    </div>
  );

  return (
    <div className="flex h-screen bg-slate-950 text-slate-200 font-sans selection:bg-red-500/30">
      
      {/* Toast Notification */}
      {notification && (
        <div className={`fixed top-4 right-4 z-50 px-6 py-3 rounded-lg shadow-xl border flex items-center gap-3 animate-in slide-in-from-right ${
          notification.type === 'error' 
            ? 'bg-red-950/90 border-red-500/50 text-red-200' 
            : 'bg-emerald-950/90 border-emerald-500/50 text-emerald-200'
        }`}>
          {notification.type === 'error' ? <Zap size={18} /> : <Check size={18} />}
          <span className="font-medium text-sm">{notification.message}</span>
        </div>
      )}

      {/* Sidebar */}
      <div className="w-20 md:w-64 bg-slate-900 border-r border-slate-800 flex flex-col justify-between flex-shrink-0">
        <div className="p-4 md:p-6">
          <div className="flex items-center gap-3 mb-10 text-red-500">
            <div className="w-8 h-8 bg-red-600 rounded-lg flex items-center justify-center text-white shadow-lg shadow-red-900/50">
              <Youtube size={20} fill="currentColor" />
            </div>
            <span className="font-bold text-xl tracking-tight text-white hidden md:block">ShortsAI</span>
          </div>

          <nav className="space-y-2">
            <SidebarItem id="trends" icon={TrendingUp} label={<span className="hidden md:inline">Trending</span>} />
            <SidebarItem id="studio" icon={Layout} label={<span className="hidden md:inline">Studio</span>} />
            <SidebarItem id="uploads" icon={Upload} label={<span className="hidden md:inline">Uploads</span>} />
            <SidebarItem id="settings" icon={Settings} label={<span className="hidden md:inline">Settings</span>} />
          </nav>
        </div>

        <div className="p-4 md:p-6 border-t border-slate-800">
          <div className="bg-slate-800 rounded-xl p-4 hidden md:block">
            <p className="text-xs text-slate-400 mb-2">Credits Remaining</p>
            <div className="w-full h-2 bg-slate-700 rounded-full overflow-hidden mb-2">
              <div className="w-[75%] h-full bg-gradient-to-r from-red-500 to-orange-500"></div>
            </div>
            <div className="flex justify-between text-xs font-medium">
              <span className="text-white">75</span>
              <span className="text-slate-500">/ 100</span>
            </div>
          </div>
        </div>
      </div>

      {/* Main Content */}
      <div className="flex-1 overflow-y-auto">
        <header className="h-16 border-b border-slate-800 flex items-center justify-between px-6 md:px-10 bg-slate-950/50 backdrop-blur sticky top-0 z-40">
          <h1 className="font-bold text-lg capitalize text-white">
            {activeTab === 'trends' ? 'Market Research' : activeTab}
          </h1>
          <div className="flex items-center gap-4">
             <div className="hidden md:flex items-center gap-2 px-3 py-1.5 bg-slate-900 rounded-full border border-slate-800">
                <div className="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></div>
                <span className="text-xs font-medium text-slate-400">System Online</span>
             </div>
             <div className="w-8 h-8 rounded-full bg-gradient-to-tr from-red-500 to-purple-600 border-2 border-slate-900"></div>
          </div>
        </header>

        <main className="p-6 md:p-10 max-w-7xl mx-auto">
          {activeTab === 'trends' && <TrendsView />}
          {activeTab === 'studio' && <StudioView />}
          {activeTab === 'uploads' && <UploadsView />}
          {activeTab === 'settings' && (
             <div className="max-w-xl">
               <h2 className="text-2xl font-bold text-white mb-6">Settings</h2>
               <div className="bg-slate-800 p-6 rounded-2xl border border-slate-700">
                  <label className="block text-sm font-medium text-slate-400 mb-2">Gemini API Key</label>
                  <input 
                    type="password" 
                    placeholder="Enter key to enable real AI generation..."
                    value={apiKey}
                    onChange={(e) => setApiKey(e.target.value)}
                    className="w-full bg-slate-900 border border-slate-700 rounded-lg px-4 py-3 text-white focus:outline-none focus:border-red-500 transition-colors"
                  />
                  <p className="text-xs text-slate-500 mt-2">Required for real-time script and image generation. Leave empty to use mock data for demo.</p>
               </div>
             </div>
          )}
        </main>
      </div>
    </div>
  );
}
