import React, { useState, useEffect } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { base44 } from '@/api/base44Client';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Link } from 'react-router-dom';
import { createPageUrl } from '@/utils';
import { 
  Sparkles, Shield, Zap, Globe, History, Calculator as CalcIcon, 
  StickyNote, User, LogOut, Brain, Microscope, MessageCircle,
  Wand2, FileText, Settings, Crown, Gamepad2, ImagePlus, Minimize2
} from 'lucide-react';
import { SoundProvider, useSound } from '@/components/audio/SoundManager';
import { EasterEggProvider, useEasterEgg } from '@/components/eastereggs/EasterEggManager';
import { SecurityProvider } from '@/components/security/SecuritySystem';
import { SplitScreenProvider, SplitScreenControls, useSplitScreen } from '@/components/splitscreen/SplitScreenManager';
import SplitSearchView from '@/components/splitscreen/SplitSearchView.jsx';
import { MiniPlayerProvider, useMiniPlayer, MiniSearchResult } from '@/components/miniplayer/MiniPlayer';
import { Button } from '@/components/ui/button';
import MobileMenu from '@/components/ui/MobileMenu';
// CursorGlow removido para melhor performance
import AIImageGenerator from '@/components/ai/AIImageGenerator';
import TabsBar from '@/components/tabs/TabsBar';
import MindMapEditor from '@/components/editor/MindMapEditor';
import CalendarPanel from '@/components/calendar/CalendarPanel';
import AdminPanel from '@/components/admin/AdminPanel';
import SearchInput from '@/components/search/SearchInput';
import SearchFilters from '@/components/search/SearchFilters';
import SearchResults from '@/components/search/SearchResults';
import ImageResults from '@/components/search/ImageResults';
import NewsResults from '@/components/search/NewsResults';
import VideoResults from '@/components/search/VideoResults';
import SmartBlocks from '@/components/search/SmartBlocks';
import TurboMode from '@/components/search/TurboMode';
import ContextualActions from '@/components/search/ContextualActions';
import ModeSelector from '@/components/search/ModeSelector';
import VisualSearch from '@/components/search/VisualSearch';
import PrivateModeIndicator, { PrivateModeOverlay } from '@/components/search/PrivateMode';
import Calculator from '@/components/calculator/Calculator';
import NotesPanel from '@/components/notes/NotesPanel';
import SearchHistoryPanel from '@/components/history/SearchHistoryPanel';
import AIPersonaCard from '@/components/ai/AIPersonaCard';
import EmotionalIndicator, { detectEmotion } from '@/components/search/EmotionalDetector';
import ExpertModeIndicator, { detectExpertMode } from '@/components/search/ExpertMode';
import FactChecker from '@/components/search/FactChecker';
import ContentEditor from '@/components/search/ContentEditor';
import WisdomPanel from '@/components/wisdom/WisdomPanel';
import COM7Chat from '@/components/com7/COM7Chat';
import AISupport from '@/components/features/AISupport';
import StudyMaterials from '@/components/features/StudyMaterials';
import UserTitleBadge from '@/components/features/UserTitles';
import AgeVerification from '@/components/features/AgeVerification';
import TextTranslator from '@/components/features/TextTranslator';
import { UltraLightWrapper, UltraLightIndicator } from '@/components/features/UltraLightMode';

function HomeContent() {
  const [query, setQuery] = useState("");
  const [hasSearched, setHasSearched] = useState(false);
  const [isSearching, setIsSearching] = useState(false);
  const [aiResponse, setAiResponse] = useState("");
  const [deepAnalysis, setDeepAnalysis] = useState(null);
  const [sources, setSources] = useState([]);
  const [searchResults, setSearchResults] = useState([]);
  const [imageResults, setImageResults] = useState([]);
  const [newsResults, setNewsResults] = useState([]);
  const [videoResults, setVideoResults] = useState([]);
  const [contextualActions, setContextualActions] = useState([]);
  const [turboPredictions, setTurboPredictions] = useState([]);
  const [activeFilter, setActiveFilter] = useState('all');
  const [searchMode, setSearchMode] = useState('quick');
  const [turboMode, setTurboMode] = useState(true);
  const [privateMode, setPrivateMode] = useState(false);
  const [showCalculator, setShowCalculator] = useState(false);
  const [showNotes, setShowNotes] = useState(false);
  const [showHistory, setShowHistory] = useState(false);
  const [showVisualSearch, setShowVisualSearch] = useState(false);
  const [showContentEditor, setShowContentEditor] = useState(false);
  const [showWisdomPanel, setShowWisdomPanel] = useState(false);
  const [showCOM7, setShowCOM7] = useState(false);
  const [showGamesMenu, setShowGamesMenu] = useState(false);
  const [showImageGenerator, setShowImageGenerator] = useState(false);
  const [showMindMap, setShowMindMap] = useState(false);
  const [showCalendar, setShowCalendar] = useState(false);
  const [showAdminPanel, setShowAdminPanel] = useState(false);
  const [showAISupport, setShowAISupport] = useState(false);
  const [ultraLightMode, setUltraLightMode] = useState(false);
  const [selectedText, setSelectedText] = useState(null);
  const [translatorPosition, setTranslatorPosition] = useState({ x: 0, y: 0 });
  const [showTranslator, setShowTranslator] = useState(false);
  const [ageVerified, setAgeVerified] = useState(false);
  const [isAdultUser, setIsAdultUser] = useState(false);
  const [sessionStartTime] = useState(Date.now());
  const [detectedEmotion, setDetectedEmotion] = useState('neutral');

  // Sistema de abas
  const [tabs, setTabs] = useState([{ id: '1', title: 'Nova aba', query: '', response: '' }]);
  const [activeTabId, setActiveTabId] = useState('1');
  const [expertMode, setExpertMode] = useState('geral');
  const [factCheck, setFactCheck] = useState(null);
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [banInfo, setBanInfo] = useState(null);

  const queryClient = useQueryClient();
  const { playSound } = useSound();
  const { checkEasterEgg, openGame } = useEasterEgg();
  const { addPlayer } = useMiniPlayer();
  const { layout: splitLayout } = useSplitScreen();

  useEffect(() => {
    const checkAuth = async () => {
      const authenticated = await base44.auth.isAuthenticated();
      setIsAuthenticated(authenticated);
      if (authenticated) {
        const currentUser = await base44.auth.me();
        setUser(currentUser);
        
        // Check if user is banned
        try {
          const bans = await base44.entities.BannedUser.filter({ 
            user_email: currentUser.email, 
            is_active: true 
          });
          
          if (bans.length > 0) {
            const activeBan = bans.find(ban => {
              if (ban.ban_type === 'permanent') return true;
              if (ban.ban_end_date) {
                return new Date(ban.ban_end_date) > new Date();
              }
              return false;
            });
            
            if (activeBan) {
              setBanInfo(activeBan);
            }
          }
        } catch (error) {
          console.error('Error checking ban status:', error);
        }
      }
    };
    checkAuth();
  }, []);

  const { data: userProfiles = [] } = useQuery({
    queryKey: ['userProfile'],
    queryFn: () => base44.entities.UserProfile.filter({ created_by: user?.email }),
    enabled: !!user?.email,
  });

  const userProfile = userProfiles[0];

  // Track usage time
  useEffect(() => {
    if (!user?.email || !userProfile) return;
    
    const interval = setInterval(() => {
      const minutesPassed = (Date.now() - sessionStartTime) / 60000;
      const today = new Date().toISOString().split('T')[0];
      
      // Reset daily time if new day
      const isNewDay = userProfile.last_active_date !== today;
      
      updateProfileMutation.mutate({
        total_time_minutes: (userProfile.total_time_minutes || 0) + 1,
        daily_time_minutes: isNewDay ? 1 : (userProfile.daily_time_minutes || 0) + 1,
        last_active: new Date().toISOString(),
        last_active_date: today
      });
    }, 60000); // Update every minute
    
    return () => clearInterval(interval);
  }, [user?.email, userProfile?.id]);

  // Text selection handler for translator
  useEffect(() => {
    const handleSelection = () => {
      const selection = window.getSelection();
      const text = selection?.toString().trim();
      
      if (text && text.length > 2 && text.length < 500) {
        const range = selection.getRangeAt(0);
        const rect = range.getBoundingClientRect();
        setSelectedText(text);
        setTranslatorPosition({ x: rect.left, y: rect.bottom });
        setShowTranslator(true);
      }
    };

    document.addEventListener('mouseup', handleSelection);
    return () => document.removeEventListener('mouseup', handleSelection);
  }, []);

  // Age verification check
  useEffect(() => {
    if (userProfile?.age_verified) {
      setAgeVerified(true);
      setIsAdultUser(userProfile.is_adult);
    }
  }, [userProfile]);

  // Check if user is admin
  const { data: adminUsers = [] } = useQuery({
    queryKey: ['adminCheck'],
    queryFn: () => base44.entities.AdminUser.list(),
    enabled: !!user?.email,
  });

  const DEVELOPER_EMAIL = 'davibergman8@gmail.com';
  const isAdmin = user?.email === DEVELOPER_EMAIL || adminUsers.some(a => a.user_email === user?.email && a.is_active);

  const { data: searchHistory = [] } = useQuery({
    queryKey: ['searchHistory'],
    queryFn: () => base44.entities.SearchHistory.list('-created_date', 100),
    enabled: isAuthenticated && !privateMode,
  });

  const updateProfileMutation = useMutation({
    mutationFn: async (data) => {
      if (userProfile?.id) {
        return base44.entities.UserProfile.update(userProfile.id, data);
      } else {
        return base44.entities.UserProfile.create(data);
      }
    },
    onSuccess: () => queryClient.invalidateQueries(['userProfile']),
  });

  const saveHistoryMutation = useMutation({
    mutationFn: (data) => base44.entities.SearchHistory.create(data),
    onSuccess: () => queryClient.invalidateQueries(['searchHistory']),
  });

  const handleSearch = async (searchQuery) => {
    // Check for easter eggs first
    if (checkEasterEgg(searchQuery)) {
      return; // Easter egg activated, don't search
    }

    playSound('search');
    setQuery(searchQuery);
    setHasSearched(true);
    setIsSearching(true);
    setAiResponse("");
    setDeepAnalysis(null);
    setSources([]);
    setSearchResults([]);
    setImageResults([]);
    setNewsResults([]);
    setVideoResults([]);
    setContextualActions([]);
    setTurboPredictions([]);

    // Detect emotion and expert mode
    const emotion = detectEmotion(searchQuery);
    const expert = detectExpertMode(searchQuery);
    setDetectedEmotion(emotion);
    setExpertMode(expert);

    try {
      // Build smart, context-aware prompt
      const userInterests = userProfile?.interests?.join(', ') || '';
      const favoriteTopics = userProfile?.favorite_topics?.map(t => t.topic).join(', ') || '';
      const recentSearches = searchHistory?.slice(0, 5).map(h => h.query).join('; ') || '';
      const learningStyle = userProfile?.personality_traits?.learning_style || 'visual';
      const preferredLength = userProfile?.personality_traits?.preferred_response_length || 'medium';
      
      let personalizedPrompt = `Você é Clarus, uma IA de busca premium com inteligência de nível GPT-4. Você é extremamente preciso, rápido e sofisticado.`;
      
      if (userProfile?.ai_persona) {
        personalizedPrompt = `Você é ${userProfile.ai_persona.name}, uma IA premium de elite. Tom: ${userProfile.ai_persona.tone}. Expertise: ${userProfile.ai_persona.expertise_areas?.join(', ')}.`;
      }
      
      // Add learning context
      const learningContext = userProfile?.settings?.aiLearning !== false ? `
CONTEXTO DE APRENDIZADO DO USUÁRIO:
- Interesses conhecidos: ${userInterests || 'não definidos'}
- Tópicos favoritos: ${favoriteTopics || 'não definidos'}
- Pesquisas recentes: ${recentSearches || 'nenhuma'}
- Estilo de aprendizado preferido: ${learningStyle}
- Tamanho de resposta preferido: ${preferredLength}

Use esse contexto para personalizar suas respostas e ser mais relevante para o usuário.` : '';

      const modeInstructions = {
        quick: "Seja direto e conciso. Responda em no máximo 3 parágrafos.",
        deep: "Faça uma análise completa e detalhada. Inclua contexto, explicações e nuances.",
        investigator: `Analise o tema de múltiplas perspectivas. Forneça:
- Um resumo neutro (imparcial)
- Um resumo técnico (detalhes especializados)
- O contexto histórico do tema`
      };

      const response = await base44.integrations.Core.InvokeLLM({
        prompt: `${personalizedPrompt}
${learningContext}

MODO: ${searchMode.toUpperCase()}
${modeInstructions[searchMode]}

Pesquisa: "${searchQuery}"

INSTRUÇÕES:
1. Responda de forma clara e útil sobre o tema pesquisado.
2. Forneça uma resposta completa em ai_response.
3. Liste algumas fontes relevantes em sources.
4. Sugira ações úteis em contextual_actions.
5. Preveja próximas pesquisas em turbo_predictions.`,
        add_context_from_internet: true,
        response_json_schema: {
          type: "object",
          properties: {
            ai_response: { type: "string", description: "Resposta principal da IA" },
            category: { type: "string", description: "Categoria da pesquisa" },
            sources: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  title: { type: "string" },
                  url: { type: "string" },
                  domain: { type: "string" }
                }
              }
            },
            contextual_actions: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  type: { type: "string" },
                  label: { type: "string" },
                  query: { type: "string" }
                }
              }
            },
            turbo_predictions: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  query: { type: "string" },
                  reason: { type: "string" }
                }
              }
            }
          }
        }
      });

      setAiResponse(response.ai_response || "Não foi possível gerar uma resposta.");
      updateCurrentTabTitle(searchQuery);
      setDeepAnalysis(null);
      setSources(response.sources || []);
      setSearchResults([]);
      setImageResults([]);
      setNewsResults([]);
      setVideoResults([]);
      setContextualActions(response.contextual_actions || []);
      setTurboPredictions(response.turbo_predictions || []);
      setFactCheck(null);

      // Save history if not private mode
      if (isAuthenticated && !privateMode) {
        await saveHistoryMutation.mutateAsync({
          query: searchQuery,
          ai_response: response.ai_response,
          category: response.category,
          search_date: new Date().toISOString()
        });
      }

    } catch (error) {
      console.error('Search error:', error);
      setAiResponse("Desculpe, ocorreu um erro ao processar sua pesquisa. Por favor, tente novamente. Erro: " + (error?.message || 'Desconhecido'));
    } finally {
      setIsSearching(false);
    }
  };

  const handleFollowUp = async (followUpQuery) => {
    const fullQuery = `${query} - ${followUpQuery}`;
    handleSearch(fullQuery);
  };

  const handleActionClick = (action) => {
    handleSearch(action.query || action.label);
  };

  // Add current search to mini player
  const handleAddToMiniPlayer = () => {
    if (aiResponse && query) {
      addPlayer(
        <MiniSearchResult query={query} result={aiResponse} />,
        query.slice(0, 30),
        { size: { width: 300, height: 180 } }
      );
    }
  };

  const handleLogin = () => base44.auth.redirectToLogin(window.location.href);
  const handleLogout = () => base44.auth.logout();

  const resetSearch = () => {
    setHasSearched(false);
    setQuery("");
    setAiResponse("");
    setDeepAnalysis(null);
    setSources([]);
    setSearchResults([]);
    setImageResults([]);
    setNewsResults([]);
    setVideoResults([]);
    setContextualActions([]);
    setTurboPredictions([]);
  };

  // Funções de abas
  const handleNewTab = () => {
    const newId = Date.now().toString();
    setTabs(prev => [...prev, { id: newId, title: 'Nova aba', query: '', response: '' }]);
    setActiveTabId(newId);
    resetSearch();
  };

  const handleCloseTab = (tabId) => {
    if (tabs.length === 1) return;
    const newTabs = tabs.filter(t => t.id !== tabId);
    setTabs(newTabs);
    if (activeTabId === tabId) {
      setActiveTabId(newTabs[newTabs.length - 1].id);
      resetSearch();
    }
  };

  const handleTabChange = (tabId) => {
    // Salvar estado atual da aba
    setTabs(prev => prev.map(t => 
      t.id === activeTabId ? { ...t, query, title: query || 'Nova aba', response: aiResponse } : t
    ));

    // Carregar nova aba
    const newTab = tabs.find(t => t.id === tabId);
    if (newTab) {
      setActiveTabId(tabId);
      setQuery(newTab.query);
      setAiResponse(newTab.response);
      setHasSearched(!!newTab.query);
    }
  };

  // Atualizar título da aba quando pesquisar
  const updateCurrentTabTitle = (title) => {
    setTabs(prev => prev.map(t => 
      t.id === activeTabId ? { ...t, title: title.slice(0, 30) || 'Nova aba', query: title } : t
    ));
  };

  const quickSearches = userProfile?.favorite_topics?.slice(0, 4).map(t => `Mais sobre ${t.topic}`) || [
    "O que é inteligência artificial?",
    "Dicas de produtividade",
    "Receitas saudáveis",
    "Notícias de tecnologia"
  ];

  // Se tiver layout dividido, renderiza o SplitSearchView
  if (splitLayout !== 'single') {
    return (
      <SplitSearchView 
        layout={splitLayout}
        privateMode={privateMode}
        searchMode={searchMode}
        turboMode={turboMode}
        isAuthenticated={isAuthenticated}
        user={user}
        userProfile={userProfile}
      />
    );
  }

  // Tela de banimento
  if (banInfo) {
    const isPermanent = banInfo.ban_type === 'permanent';
    const banEndDate = banInfo.ban_end_date ? new Date(banInfo.ban_end_date) : null;
    const timeRemaining = banEndDate ? Math.max(0, banEndDate - new Date()) : null;
    const daysRemaining = timeRemaining ? Math.ceil(timeRemaining / (1000 * 60 * 60 * 24)) : null;
    
    return (
      <div className="min-h-screen bg-gradient-to-br from-red-950 via-slate-950 to-red-950 flex items-center justify-center p-4">
        <motion.div
          initial={{ scale: 0.9, opacity: 0 }}
          animate={{ scale: 1, opacity: 1 }}
          className="bg-slate-900 border border-red-500/50 rounded-2xl p-8 max-w-md w-full text-center shadow-2xl shadow-red-500/20"
        >
          <div className="w-20 h-20 bg-red-500/20 rounded-full flex items-center justify-center mx-auto mb-6">
            <Shield className="w-10 h-10 text-red-500" />
          </div>
          <h1 className="text-2xl font-bold text-white mb-2">Conta Suspensa</h1>
          <p className="text-slate-400 mb-6">
            Sua conta foi {isPermanent ? 'permanentemente banida' : 'temporariamente suspensa'}.
          </p>
          
          <div className="bg-red-500/10 border border-red-500/30 rounded-xl p-4 mb-6">
            <p className="text-sm text-slate-300 mb-2">
              <span className="text-red-400 font-semibold">Motivo:</span> {banInfo.reason}
            </p>
            {!isPermanent && daysRemaining !== null && (
              <p className="text-sm text-slate-300">
                <span className="text-yellow-400 font-semibold">Tempo restante:</span>{' '}
                {daysRemaining > 0 ? `${daysRemaining} dia${daysRemaining > 1 ? 's' : ''}` : 'Menos de 1 dia'}
              </p>
            )}
            {isPermanent && (
              <p className="text-sm text-red-400 font-semibold">
                Banimento permanente
              </p>
            )}
          </div>

          <p className="text-xs text-slate-500 mb-4">
            Se você acredita que isso é um erro, entre em contato com o suporte.
          </p>
          
          <Button 
            onClick={() => base44.auth.logout()}
            variant="outline"
            className="border-red-500/50 text-red-400 hover:bg-red-500/20"
          >
            <LogOut className="w-4 h-4 mr-2" />
            Sair
          </Button>
        </motion.div>
      </div>
    );
  }

  return (
    <div className={`min-h-screen transition-all duration-500 flex flex-col ${
      privateMode 
        ? 'bg-transparent' 
        : 'bg-gradient-to-br from-slate-950 via-purple-950 to-slate-950'
    }`}>
      {/* Tabs Bar */}
      <TabsBar
        tabs={tabs}
        activeTab={activeTabId}
        onTabChange={handleTabChange}
        onNewTab={handleNewTab}
        onCloseTab={handleCloseTab}
      />
      
      {/* Private Mode Overlay */}
      <PrivateModeOverlay isActive={privateMode} />

      {/* Header Tools - Menu Hamburger */}
      <div className="fixed top-4 right-4 z-40 flex items-center gap-2">
        <PrivateModeIndicator isActive={privateMode} onToggle={() => {
                setPrivateMode(!privateMode);
                playSound('privateMode');
              }} />
        
        <MobileMenu
          isAuthenticated={isAuthenticated}
          user={user}
          userProfile={userProfile}
          onShowHistory={() => setShowHistory(true)}
          onShowCalculator={() => setShowCalculator(true)}
          onShowNotes={() => setShowNotes(true)}
          onShowWisdom={() => setShowWisdomPanel(true)}
          onShowCOM7={() => setShowCOM7(true)}
          onShowGames={() => setShowGamesMenu(true)}
          onShowImageGenerator={() => setShowImageGenerator(true)}
          onShowMindMap={() => setShowMindMap(true)}
          onShowCalendar={() => setShowCalendar(true)}
          onShowAdmin={() => setShowAdminPanel(true)}
          onShowAISupport={() => setShowAISupport(true)}
          isAdmin={isAdmin}
          ultraLightMode={ultraLightMode}
          onToggleUltraLight={() => setUltraLightMode(!ultraLightMode)}
          onLogin={handleLogin}
          onLogout={handleLogout}
        />
      </div>

      <div className="max-w-4xl mx-auto px-3 sm:px-6">
        <AnimatePresence mode="wait">
          {!hasSearched ? (
            <motion.div
              key="landing"
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0, y: -20 }}
              className="flex flex-col items-center justify-center min-h-screen py-8 sm:py-12"
            >
              {/* Logo Premium/Luxury - sempre visível */}
              <motion.div initial={{ scale: 0.8, opacity: 0 }} animate={{ scale: 1, opacity: 1 }} transition={{ delay: 0.1 }} className="mb-4 sm:mb-6">
                <div className="flex flex-col sm:flex-row items-center gap-3 sm:gap-4">
                  <div className="relative">
                    <div className="absolute inset-0 rounded-2xl sm:rounded-3xl blur-xl opacity-70 animate-pulse bg-gradient-to-br from-purple-500 via-pink-500 to-purple-600" />
                    <div className="relative p-4 sm:p-5 rounded-2xl sm:rounded-3xl shadow-2xl bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 border border-purple-500/40">
                      <div className="relative">
                        <Crown className="w-8 h-8 sm:w-10 sm:h-10 text-purple-400" />
                        <Sparkles className="w-4 h-4 sm:w-5 sm:h-5 absolute -top-1 -right-1 animate-pulse text-pink-400" />
                      </div>
                    </div>
                  </div>
                  <div className="text-center sm:text-left">
                    <h1 className="text-3xl sm:text-5xl font-bold tracking-tight bg-gradient-to-r from-purple-300 via-pink-300 to-purple-300 bg-clip-text text-transparent">
                      Clarus
                    </h1>
                    <p className="text-xs sm:text-sm font-medium tracking-[0.2em] sm:tracking-[0.3em] uppercase text-purple-400/80">Premium Intelligence</p>
                  </div>
                </div>
              </motion.div>

              {!privateMode && (
                <>
                  {/* AI Persona */}
                  {isAuthenticated && userProfile?.ai_persona && (
                    <motion.div initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} transition={{ delay: 0.15 }} className="mb-4 sm:mb-6 w-full max-w-md px-2">
                      <AIPersonaCard userProfile={userProfile} compact />
                    </motion.div>
                  )}

                  {/* Tagline */}
                  <motion.p initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} transition={{ delay: 0.2 }} className="text-sm sm:text-lg mb-4 sm:mb-6 text-center text-purple-300/70 px-4">
                    {userProfile?.ai_persona?.greeting || 'Pesquisa + Sabedoria + Chat em uma porrada só'}
                  </motion.p>

                  {/* Mode Selector */}
                  <motion.div initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} transition={{ delay: 0.25 }} className="mb-4 sm:mb-6 w-full overflow-x-auto px-2">
                    <ModeSelector activeMode={searchMode} onModeChange={setSearchMode} />
                  </motion.div>
                </>
              )}

              {/* Search Input - with glow */}
              <motion.div 
                initial={{ opacity: 0, y: 20 }} 
                animate={{ opacity: 1, y: 0 }} 
                transition={{ delay: privateMode ? 0 : 0.3 }} 
                className={`w-full ${privateMode ? 'mb-0' : 'mb-6 sm:mb-8'} relative px-2`}
              >
                <div className="absolute inset-0 bg-purple-500/20 rounded-3xl blur-2xl scale-110" />
                <div className="relative">
                  <SearchInput 
                    onSearch={handleSearch} 
                    isSearching={isSearching}
                    onVisualSearch={() => setShowVisualSearch(true)}
                    privateMode={privateMode}
                  />
                </div>
              </motion.div>

              {!privateMode && (
                <>
                  {/* Image Generator Button */}
                  <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ delay: 0.35 }} className="mb-4 sm:mb-6">
                    <Button
                      onClick={() => setShowImageGenerator(true)}
                      className="bg-gradient-to-r from-pink-600 via-purple-600 to-pink-600 hover:from-pink-700 hover:via-purple-700 hover:to-pink-700 text-white shadow-lg shadow-purple-500/30 rounded-full px-6"
                    >
                      <ImagePlus className="w-4 h-4 mr-2" />
                      Gerar Imagem com IA
                    </Button>
                  </motion.div>

                  {/* Quick Searches */}
                  <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ delay: 0.4 }} className="flex flex-wrap justify-center gap-2 mb-8 sm:mb-12 px-2">
                    {quickSearches.map((term, index) => (
                      <button
                        key={index}
                        onClick={() => handleSearch(term)}
                        className="px-3 sm:px-4 py-2 text-xs sm:text-sm rounded-full transition-all text-purple-200 bg-white/5 border border-purple-500/30 hover:border-purple-400 hover:text-white hover:bg-purple-500/20 backdrop-blur-sm"
                      >
                        {term}
                      </button>
                    ))}
                  </motion.div>

                  {/* Features - Luxury Cards */}
                  <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} transition={{ delay: 0.5 }} className="grid grid-cols-1 sm:grid-cols-3 gap-3 sm:gap-4 w-full max-w-2xl px-2">
                    <div className="flex flex-col items-center text-center p-4 sm:p-5 rounded-2xl backdrop-blur-xl border transition-all hover:scale-105 bg-white/5 border-purple-500/20 hover:border-green-400/50 hover:shadow-lg hover:shadow-green-500/20">
                      <div className="p-3 bg-gradient-to-br from-green-400 to-emerald-600 rounded-xl mb-3 shadow-lg shadow-green-500/30">
                        <Zap className="w-5 h-5 text-white" />
                      </div>
                      <h3 className="font-medium mb-1 text-white">Modo Rápido</h3>
                      <p className="text-xs text-purple-300/60">Respostas em &lt;1s</p>
                    </div>

                    <div className="flex flex-col items-center text-center p-4 sm:p-5 rounded-2xl backdrop-blur-xl border transition-all hover:scale-105 bg-white/5 border-purple-500/20 hover:border-blue-400/50 hover:shadow-lg hover:shadow-blue-500/20">
                      <div className="p-3 bg-gradient-to-br from-blue-400 to-indigo-600 rounded-xl mb-3 shadow-lg shadow-blue-500/30">
                        <Brain className="w-5 h-5 text-white" />
                      </div>
                      <h3 className="font-medium mb-1 text-white">Modo Profundo</h3>
                      <p className="text-xs text-purple-300/60">Análise completa</p>
                    </div>

                    <div className="flex flex-col items-center text-center p-4 sm:p-5 rounded-2xl backdrop-blur-xl border transition-all hover:scale-105 bg-white/5 border-purple-500/20 hover:border-pink-400/50 hover:shadow-lg hover:shadow-pink-500/20">
                      <div className="p-3 bg-gradient-to-br from-purple-400 to-pink-600 rounded-xl mb-3 shadow-lg shadow-purple-500/30">
                        <Microscope className="w-5 h-5 text-white" />
                      </div>
                      <h3 className="font-medium mb-1 text-white">Investigador</h3>
                      <p className="text-xs text-purple-300/60">Múltiplas perspectivas</p>
                    </div>
                  </motion.div>

                  {!isAuthenticated && (
                    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ delay: 0.6 }} className="mt-6 sm:mt-8 text-center">
                      <p className="text-sm text-purple-300/60 mb-3">Entre para ter IA personalizada e histórico</p>
                      <Button onClick={handleLogin} variant="outline" className="border-purple-500/50 text-purple-300 hover:bg-purple-500/20">
                        <User className="w-4 h-4 mr-2" />
                        Criar conta gratuita
                      </Button>
                    </motion.div>
                  )}
                </>
              )}
            </motion.div>
          ) : (
            <motion.div key="results" initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="py-4 sm:py-6">
              {/* Header */}
              <div className="sticky top-0 pt-2 sm:pt-4 pb-4 sm:pb-6 z-10 bg-gradient-to-b from-slate-950 via-slate-950/95 to-transparent">
                <div className="flex items-center justify-between gap-2 sm:gap-4 mb-3 sm:mb-4">
                  <button onClick={resetSearch} className="flex items-center gap-2 sm:gap-3 group">
                    <div className="relative">
                      <div className="absolute inset-0 bg-gradient-to-br from-purple-400 to-pink-600 rounded-xl blur opacity-50" />
                      <div className="relative p-2 bg-gradient-to-br from-slate-900 to-purple-900 rounded-xl border border-purple-500/30">
                        <Crown className="w-4 h-4 sm:w-5 sm:h-5 text-purple-400" />
                      </div>
                    </div>
                    <span className="text-lg sm:text-xl font-bold transition-colors text-white group-hover:text-purple-400">
                      Clarus
                    </span>
                  </button>
                  
                  <ModeSelector activeMode={searchMode} onModeChange={setSearchMode} />
                  
                  {detectedEmotion !== 'neutral' && (
                    <EmotionalIndicator emotion={detectedEmotion} compact />
                  )}
                </div>
                
                {expertMode !== 'geral' && (
                  <ExpertModeIndicator mode={expertMode} />
                )}
                
                <SearchInput 
                  onSearch={handleSearch} 
                  isSearching={isSearching} 
                  initialQuery={query}
                  onVisualSearch={() => setShowVisualSearch(true)}
                  privateMode={privateMode}
                />
                
                <div className="mt-4 flex items-center justify-between flex-wrap gap-2">
                  <SearchFilters activeFilter={activeFilter} onFilterChange={setActiveFilter} />
                  <div className="flex items-center gap-2">
                    <Button
                                              variant="outline"
                                              size="sm"
                                              onClick={() => setShowContentEditor(true)}
                                            >
                                              <Wand2 className="w-4 h-4 mr-1" />
                                              Editar
                                            </Button>
                                            {aiResponse && (
                                              <Button
                                                variant="outline"
                                                size="sm"
                                                onClick={handleAddToMiniPlayer}
                                                title="Fixar em mini player"
                                              >
                                                <Minimize2 className="w-4 h-4" />
                                              </Button>
                                            )}
                    <Button
                    variant={turboMode ? "default" : "outline"}
                    size="sm"
                    onClick={() => setTurboMode(!turboMode)}
                    className={turboMode ? "bg-amber-500 hover:bg-amber-600" : ""}
                  >
                    <Zap className="w-4 h-4 mr-1" />
                      Turbo
                    </Button>
                  </div>
                </div>
              </div>

              {/* Results */}
              <div className="space-y-6 pb-12">
                {/* Smart AI Response Block */}
                <SmartBlocks
                  aiResponse={aiResponse}
                  deepAnalysis={deepAnalysis}
                  sources={sources}
                  isLoading={isSearching}
                  onAskFollowUp={handleFollowUp}
                  mode={searchMode}
                />

                {/* Fact Checker */}
                {factCheck && !isSearching && (
                  <FactChecker factCheck={factCheck} isLoading={isSearching} />
                )}

                {/* Contextual Actions */}
                {!isSearching && contextualActions.length > 0 && (
                  <ContextualActions actions={contextualActions} onActionClick={handleActionClick} />
                )}

                {/* Study Materials */}
                <StudyMaterials query={query} isVisible={!isSearching && aiResponse} />

                {/* Turbo Mode Predictions */}
                <TurboMode
                  predictions={turboPredictions}
                  onPredictionClick={handleSearch}
                  isActive={turboMode && !isSearching}
                />

                {/* Filter Results */}
                {activeFilter === 'all' && (
                  <SearchResults results={searchResults} isLoading={isSearching} query={query} />
                )}
                {activeFilter === 'images' && (
                  <ImageResults images={imageResults} isLoading={isSearching} />
                )}
                {activeFilter === 'news' && (
                  <NewsResults news={newsResults} isLoading={isSearching} />
                )}
                {activeFilter === 'videos' && (
                  <VideoResults videos={videoResults} isLoading={isSearching} />
                )}
              </div>
            </motion.div>
          )}
        </AnimatePresence>
      </div>

      {/* Games Menu */}
      <AnimatePresence>
        {showGamesMenu && (
          <motion.div
            initial={{ opacity: 0, y: -10 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -10 }}
            className="fixed top-16 right-4 z-50 bg-white rounded-2xl shadow-2xl border border-slate-200 p-4 w-64"
          >
            <h3 className="font-bold text-slate-800 mb-3 flex items-center gap-2">
              <Gamepad2 className="w-5 h-5 text-indigo-600" />
              Easter Eggs 🎮
            </h3>
            <div className="grid grid-cols-2 gap-2">
              {[
                { id: 'snake', name: '🐍 Snake', color: 'bg-green-100 hover:bg-green-200' },
                { id: 'tictactoe', name: '⭕ Velha', color: 'bg-blue-100 hover:bg-blue-200' },
                { id: 'pong', name: '🏓 Pong', color: 'bg-purple-100 hover:bg-purple-200' },
                { id: 'memory', name: '🧠 Memória', color: 'bg-pink-100 hover:bg-pink-200' },
                { id: 'dino', name: '🦖 Dino', color: 'bg-orange-100 hover:bg-orange-200' },
                { id: 'flappy', name: '🐦 Flappy', color: 'bg-yellow-100 hover:bg-yellow-200' },
                { id: 'brick', name: '🧱 Brick', color: 'bg-red-100 hover:bg-red-200' },
                { id: 'simon', name: '🎹 Simon', color: 'bg-cyan-100 hover:bg-cyan-200' },
                { id: 'minesweeper', name: '💣 Minas', color: 'bg-slate-100 hover:bg-slate-200' },
                { id: 'tetris', name: '🧱 Tetris', color: 'bg-indigo-100 hover:bg-indigo-200' },
              ].map(game => (
                <button
                  key={game.id}
                  onClick={() => { openGame(game.id); setShowGamesMenu(false); }}
                  className={`${game.color} rounded-lg p-2 text-xs font-medium text-slate-700 transition-colors`}
                >
                  {game.name}
                </button>
              ))}
            </div>
            <p className="text-xs text-slate-400 mt-3 text-center">
              Ou pesquise: "cobra", "velha", "pong"...
            </p>
          </motion.div>
        )}
      </AnimatePresence>

      {/* Split Screen Controls */}
                  <SplitScreenControls />

                  {/* Modals */}
                  <AnimatePresence>
        {showCalculator && <Calculator isOpen={showCalculator} onClose={() => setShowCalculator(false)} />}
        {showNotes && <NotesPanel isOpen={showNotes} onClose={() => setShowNotes(false)} />}
        {showHistory && <SearchHistoryPanel isOpen={showHistory} onClose={() => setShowHistory(false)} onSearchAgain={handleSearch} />}
        {showVisualSearch && <VisualSearch isOpen={showVisualSearch} onClose={() => setShowVisualSearch(false)} onSearchResult={handleSearch} />}
        {showContentEditor && <ContentEditor isOpen={showContentEditor} onClose={() => setShowContentEditor(false)} initialContent={aiResponse} />}
        {showWisdomPanel && <WisdomPanel isOpen={showWisdomPanel} onClose={() => setShowWisdomPanel(false)} />}
        {showCOM7 && <COM7Chat isOpen={showCOM7} onClose={() => setShowCOM7(false)} />}
        {showImageGenerator && <AIImageGenerator isOpen={showImageGenerator} onClose={() => setShowImageGenerator(false)} />}
        {showMindMap && <MindMapEditor isOpen={showMindMap} onClose={() => setShowMindMap(false)} />}
        {showCalendar && <CalendarPanel isOpen={showCalendar} onClose={() => setShowCalendar(false)} />}
        {showAdminPanel && <AdminPanel isOpen={showAdminPanel} onClose={() => setShowAdminPanel(false)} currentUserEmail={user?.email} />}
        {showAISupport && <AISupport isOpen={showAISupport} onClose={() => setShowAISupport(false)} />}
      </AnimatePresence>

      {/* Text Translator */}
      <AnimatePresence>
        {showTranslator && selectedText && (
          <TextTranslator
            selectedText={selectedText}
            position={translatorPosition}
            onClose={() => {
              setShowTranslator(false);
              setSelectedText(null);
            }}
          />
        )}
      </AnimatePresence>

      {/* Age Verification Modal */}
      {isAuthenticated && userProfile && !userProfile.age_verified && (
        <AgeVerification
          userProfile={userProfile}
          onVerified={(isAdult) => {
            setAgeVerified(true);
            setIsAdultUser(isAdult);
          }}
          onUpdateProfile={(data) => updateProfileMutation.mutateAsync(data)}
        />
      )}
    </div>
  );
}

export default function Home() {
  return (
    <SecurityProvider>
      <SoundProvider>
        <EasterEggProvider>
          <SplitScreenProvider>
            <MiniPlayerProvider>
              <HomeContent />
            </MiniPlayerProvider>
          </SplitScreenProvider>
        </EasterEggProvider>
      </SoundProvider>
    </SecurityProvider>
  );
}
