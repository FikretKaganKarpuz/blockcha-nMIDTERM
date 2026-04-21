import React, { useState, useEffect } from 'react';
import { 
  Wallet, Activity, Database, ArrowRight, ShieldCheck, 
  UploadCloud, Search, CheckCircle2, History, Send 
} from 'lucide-react';

export default function NexusHealthApp() {
  // --- UYGULAMA DURUMLARI (STATES) ---
  const [walletConnected, setWalletConnected] = useState(false);
  const [walletAddress, setWalletAddress] = useState('');
  const [balance, setBalance] = useState(0);
  const [activeTab, setActiveTab] = useState('dashboard'); // dashboard, market, upload
  const [transactions, setTransactions] = useState([]);
  
  // Modal Durumları
  const [showTransferModal, setShowTransferModal] = useState(false);
  const [transferAmount, setTransferAmount] = useState('');
  const [transferTo, setTransferTo] = useState('');
  const [isProcessing, setIsProcessing] = useState(false);
  const [notification, setNotification] = useState(null);

  // --- MOCK DATA (Örnek Verisetleri) ---
  const datasets = [
    { id: 1, name: "Kardiyoloji: 10k Hasta EKG Verisi", provider: "Medipol Node", price: 250, type: "Klinik Veri" },
    { id: 2, name: "Giyilebilir Teknoloji: Uyku Döngüleri", provider: "IoT Havuzu", price: 100, type: "IoT Verisi" },
    { id: 3, name: "Diyabet Tip-2 Kan Şekeri Analizi", provider: "Global Konsorsiyum", price: 400, type: "Klinik Veri" }
  ];

  // --- FONKSİYONLAR ---

  // Bildirim gösterme fonksiyonu
  const showNotification = (msg, type = 'success') => {
    setNotification({ msg, type });
    setTimeout(() => setNotification(null), 3000);
  };

  // Cüzdan bağlama simülasyonu
  const handleConnectWallet = () => {
    setIsProcessing(true);
    setTimeout(() => {
      setWalletConnected(true);
      // Rastgele bir Web3 cüzdan adresi oluştur
      setWalletAddress('0x' + Math.random().toString(16).substr(2, 40));
      // Başlangıç bakiyesi olarak 500 NEXUS ver (Test ortamı için)
      setBalance(500);
      setIsProcessing(false);
      showNotification("Cüzdan başarıyla bağlandı!");
      
      // İlk işlemi geçmişe ekle
      addTransaction("Airdrop (Testnet)", 500, "in");
    }, 1000);
  };

  // Cüzdanı koparma
  const handleDisconnectWallet = () => {
    setWalletConnected(false);
    setWalletAddress('');
    setBalance(0);
    setTransactions([]);
    setActiveTab('dashboard');
  };

  // İşlem geçmişine kayıt ekleme
  const addTransaction = (title, amount, type) => {
    const newTx = {
      id: Math.random().toString(36).substr(2, 9),
      title,
      amount,
      type, // 'in' (gelen) veya 'out' (giden)
      date: new Date().toLocaleTimeString()
    };
    setTransactions(prev => [newTx, ...prev]);
  };

  // Token transfer / Gönderim işlemi
  const handleTransfer = (e) => {
    e.preventDefault();
    const amount = parseFloat(transferAmount);
    
    if (!walletConnected) return showNotification("Önce cüzdan bağlamalısınız!", "error");
    if (isNaN(amount) || amount <= 0) return showNotification("Geçerli bir miktar girin.", "error");
    if (amount > balance) return showNotification("Yetersiz bakiye!", "error");
    if (transferTo.length < 10) return showNotification("Geçerli bir alıcı adresi girin.", "error");

    setIsProcessing(true);
    
    // Akıllı kontrat işlem simülasyonu (2 saniye gecikme)
    setTimeout(() => {
      setBalance(prev => prev - amount);
      addTransaction(`Transfer -> ${transferTo.substring(0, 6)}...`, amount, "out");
      setIsProcessing(false);
      setShowTransferModal(false);
      setTransferAmount('');
      setTransferTo('');
      showNotification("Transfer başarıyla tamamlandı.");
    }, 2000);
  };

  // Veri Pazarı'ndan veri satın alma (Stake etme)
  const handleBuyData = (dataset) => {
    if (!walletConnected) return showNotification("Önce cüzdan bağlamalısınız!", "error");
    if (dataset.price > balance) return showNotification("Yetersiz bakiye!", "error");

    setIsProcessing(true);
    setTimeout(() => {
      setBalance(prev => prev - dataset.price);
      addTransaction(`Veri Erişimi: ${dataset.name}`, dataset.price, "out");
      setIsProcessing(false);
      showNotification(`${dataset.name} erişimi akıllı kontrat ile onaylandı!`);
    }, 1500);
  };

  // Hasta olarak sisteme veri yükleyip ödül kazanma
  const handleUploadData = (e) => {
    e.preventDefault();
    if (!walletConnected) return showNotification("Önce cüzdan bağlamalısınız!", "error");

    setIsProcessing(true);
    setTimeout(() => {
      const reward = Math.floor(Math.random() * 20) + 10; // 10 ile 30 arası ödül
      setBalance(prev => prev + reward);
      addTransaction("Veri Yükleme Ödülü (Zero-Knowledge Proof)", reward, "in");
      setIsProcessing(false);
      showNotification(`Veriniz anonimleştirildi. ${reward} NEXUS kazandınız!`);
      e.target.reset();
    }, 2000);
  };

  // --- ARAYÜZ (UI) BİLEŞENLERİ ---

  return (
    <div className="min-h-screen bg-slate-50 text-slate-800 font-sans">
      
      {/* Üst Navigasyon Çubuğu */}
      <nav className="bg-slate-900 text-white p-4 shadow-lg">
        <div className="max-w-6xl mx-auto flex justify-between items-center">
          <div className="flex items-center space-x-2">
            <Activity className="text-teal-400 h-8 w-8" />
            <span className="text-2xl font-bold tracking-tight">Nexus<span className="text-teal-400">Health</span></span>
          </div>
          
          <div className="flex items-center space-x-6">
            {walletConnected && (
              <div className="hidden md:flex space-x-4">
                <button onClick={() => setActiveTab('dashboard')} className={`hover:text-teal-400 transition ${activeTab === 'dashboard' ? 'text-teal-400' : 'text-slate-300'}`}>Kontrol Paneli</button>
                <button onClick={() => setActiveTab('market')} className={`hover:text-teal-400 transition ${activeTab === 'market' ? 'text-teal-400' : 'text-slate-300'}`}>AI Veri Pazarı</button>
                <button onClick={() => setActiveTab('upload')} className={`hover:text-teal-400 transition ${activeTab === 'upload' ? 'text-teal-400' : 'text-slate-300'}`}>Veri Yükle & Kazan</button>
              </div>
            )}
            
            {!walletConnected ? (
              <button 
                onClick={handleConnectWallet}
                disabled={isProcessing}
                className="bg-teal-500 hover:bg-teal-400 text-white px-6 py-2 rounded-full font-medium transition flex items-center space-x-2 disabled:opacity-50"
              >
                <Wallet className="w-4 h-4" />
                <span>{isProcessing ? 'Bağlanıyor...' : 'Cüzdanı Bağla'}</span>
              </button>
            ) : (
              <div className="flex items-center space-x-4">
                <div className="bg-slate-800 border border-slate-700 px-4 py-2 rounded-full flex items-center space-x-2">
                  <div className="w-2 h-2 rounded-full bg-green-400 animate-pulse"></div>
                  <span className="text-sm font-mono text-slate-300">{walletAddress.substring(0, 6)}...{walletAddress.substring(38)}</span>
                </div>
                <button onClick={handleDisconnectWallet} className="text-xs text-slate-400 hover:text-white underline">Çıkış</button>
              </div>
            )}
          </div>
        </div>
      </nav>

      {/* Bildirim Alanı (Toast) */}
      {notification && (
        <div className={`fixed top-20 right-4 p-4 rounded-lg shadow-xl z-50 flex items-center space-x-2 transform transition-all duration-300 ${notification.type === 'error' ? 'bg-red-500 text-white' : 'bg-teal-500 text-white'}`}>
          <CheckCircle2 className="w-5 h-5" />
          <p className="font-medium">{notification.msg}</p>
        </div>
      )}

      {/* Ana İçerik Alanı */}
      <main className="max-w-6xl mx-auto p-6 mt-4">
        
        {!walletConnected ? (
          // Cüzdan Bağlı Değilken Görünen Karşılama Ekranı
          <div className="text-center py-20">
            <ShieldCheck className="w-24 h-24 mx-auto text-slate-300 mb-6" />
            <h1 className="text-4xl font-bold text-slate-900 mb-4">Sağlık Verisinde Merkeziyetsiz Devrim</h1>
            <p className="text-lg text-slate-600 mb-8 max-w-2xl mx-auto">
              Araştırmacılar için güvenilir veri, hastalar için şeffaf kazanç. 
              Sistemi kullanmaya başlamak ve Web3 ekosistemine katılmak için dijital cüzdanınızı bağlayın.
            </p>
            <button 
              onClick={handleConnectWallet}
              className="bg-slate-900 hover:bg-slate-800 text-white text-lg px-8 py-4 rounded-full font-medium transition shadow-lg inline-flex items-center space-x-3"
            >
              <Wallet className="w-6 h-6" />
              <span>Web3 Cüzdanı ile Giriş Yap</span>
            </button>
          </div>
        ) : (
          // Cüzdan Bağlıyken Görünen İçerikler
          <div>
            
            {/* DİNAMİK SEKMELER (TABS) */}
            
            {/* 1. KONTROL PANELİ (DASHBOARD) */}
            {activeTab === 'dashboard' && (
              <div className="space-y-6">
                
                {/* Bakiye ve Hızlı İşlem Kartı */}
                <div className="bg-gradient-to-r from-slate-900 to-slate-800 rounded-2xl p-8 text-white shadow-xl flex flex-col md:flex-row justify-between items-center">
                  <div>
                    <p className="text-slate-400 font-medium mb-1">Kullanılabilir Bakiye</p>
                    <div className="flex items-baseline space-x-2">
                      <h2 className="text-5xl font-bold">{balance.toFixed(2)}</h2>
                      <span className="text-xl text-teal-400 font-semibold">$NEXUS</span>
                    </div>
                    <p className="text-sm text-slate-400 mt-2 flex items-center">
                      <ShieldCheck className="w-4 h-4 mr-1 text-green-400" />
                      Smart Contract Onaylı
                    </p>
                  </div>
                  <div className="mt-6 md:mt-0 flex space-x-4">
                    <button 
                      onClick={() => setShowTransferModal(true)}
                      className="bg-teal-500 hover:bg-teal-400 text-white px-6 py-3 rounded-xl font-medium transition flex items-center space-x-2 shadow-lg"
                    >
                      <Send className="w-5 h-5" />
                      <span>Token Gönder</span>
                    </button>
                  </div>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                  {/* Cüzdan Detayları */}
                  <div className="bg-white rounded-2xl p-6 shadow-sm border border-slate-100">
                    <h3 className="text-lg font-bold text-slate-800 mb-4 flex items-center">
                      <Database className="w-5 h-5 mr-2 text-teal-500" />
                      Ağ İstatistikleri
                    </h3>
                    <div className="space-y-4">
                      <div className="flex justify-between pb-3 border-b border-slate-50">
                        <span className="text-slate-500">Bağlı Ağ</span>
                        <span className="font-medium text-slate-800">Ethereum Sepolia (Testnet)</span>
                      </div>
                      <div className="flex justify-between pb-3 border-b border-slate-50">
                        <span className="text-slate-500">Ağ Komisyonu (Burn)</span>
                        <span className="font-medium text-slate-800">%2.00</span>
                      </div>
                      <div className="flex justify-between pb-3">
                        <span className="text-slate-500">Anonimlik Durumu</span>
                        <span className="font-medium text-green-500 flex items-center"><CheckCircle2 className="w-4 h-4 mr-1" /> Zero-Knowledge Aktif</span>
                      </div>
                    </div>
                  </div>

                  {/* Son İşlemler */}
                  <div className="bg-white rounded-2xl p-6 shadow-sm border border-slate-100">
                    <h3 className="text-lg font-bold text-slate-800 mb-4 flex items-center">
                      <History className="w-5 h-5 mr-2 text-teal-500" />
                      Son İşlemler (On-Chain)
                    </h3>
                    {transactions.length === 0 ? (
                      <p className="text-slate-500 text-center py-4">Henüz bir işlem bulunmuyor.</p>
                    ) : (
                      <div className="space-y-3 max-h-48 overflow-y-auto pr-2">
                        {transactions.map(tx => (
                          <div key={tx.id} className="flex justify-between items-center p-3 hover:bg-slate-50 rounded-lg transition">
                            <div className="flex flex-col">
                              <span className="font-medium text-slate-700 text-sm">{tx.title}</span>
                              <span className="text-xs text-slate-400">{tx.date} | Hash: 0x{tx.id}...</span>
                            </div>
                            <span className={`font-bold ${tx.type === 'in' ? 'text-green-500' : 'text-slate-700'}`}>
                              {tx.type === 'in' ? '+' : '-'}{tx.amount} NEXUS
                            </span>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>
                </div>
              </div>
            )}

            {/* 2. VERİ PAZARI (YAPAY ZEKA ŞİRKETLERİ İÇİN) */}
            {activeTab === 'market' && (
              <div className="space-y-6">
                <div className="mb-8">
                  <h2 className="text-2xl font-bold text-slate-900">Araştırmacı Veri Pazarı</h2>
                  <p className="text-slate-600">Makine öğrenmesi modellerinizi eğitmek için IPFS üzerinde barındırılan, etik onaylı sağlık verilerine erişin.</p>
                </div>
                
                <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
                  {datasets.map(dataset => (
                    <div key={dataset.id} className="bg-white rounded-2xl p-6 shadow-sm border border-slate-200 hover:border-teal-500 transition-colors flex flex-col h-full">
                      <div className="mb-4 flex justify-between items-start">
                        <span className="bg-slate-100 text-slate-600 text-xs font-bold px-3 py-1 rounded-full uppercase tracking-wider">
                          {dataset.type}
                        </span>
                        <div className="bg-teal-50 text-teal-700 font-bold px-3 py-1 rounded-lg">
                          {dataset.price} NEXUS
                        </div>
                      </div>
                      <h3 className="text-xl font-bold text-slate-800 mb-2">{dataset.name}</h3>
                      <p className="text-sm text-slate-500 mb-6 flex-grow">
                        Sağlayıcı: {dataset.provider}<br/>
                        Şifreleme: AES-256 (IPFS Node)<br/>
                        <span className="text-green-600 flex items-center mt-1"><CheckCircle2 className="w-3 h-3 mr-1"/> HIPAA & KVKK Uyumlu</span>
                      </p>
                      <button 
                        onClick={() => handleBuyData(dataset)}
                        disabled={isProcessing}
                        className="w-full bg-slate-900 hover:bg-slate-800 text-white py-3 rounded-xl font-medium transition flex items-center justify-center space-x-2 disabled:opacity-50"
                      >
                        <Search className="w-4 h-4" />
                        <span>Veriye Erişimi Aç</span>
                      </button>
                    </div>
                  ))}
                </div>
              </div>
            )}

            {/* 3. VERİ YÜKLEME (HASTALAR İÇİN) */}
            {activeTab === 'upload' && (
              <div className="max-w-2xl mx-auto">
                <div className="bg-white rounded-2xl shadow-xl overflow-hidden">
                  <div className="bg-gradient-to-r from-teal-500 to-teal-400 p-8 text-white text-center">
                    <UploadCloud className="w-16 h-16 mx-auto mb-4 opacity-90" />
                    <h2 className="text-2xl font-bold mb-2">Verinizi Yükleyin, Ödül Kazanın</h2>
                    <p className="opacity-90 text-sm">Hiçbir kişisel veriniz (isim, TC, adres) kaydedilmez. Akıllı kontrat veriyi maskeler.</p>
                  </div>
                  
                  <form onSubmit={handleUploadData} className="p-8 space-y-6">
                    <div>
                      <label className="block text-sm font-medium text-slate-700 mb-2">Veri Tipi Seçin</label>
                      <select className="w-full border border-slate-300 rounded-xl px-4 py-3 focus:outline-none focus:ring-2 focus:ring-teal-500 focus:border-transparent">
                        <option>Akıllı Saat Nabız / EKG Özeti</option>
                        <option>Uyku Döngüsü (Apple Health/Fitbit)</option>
                        <option>Anonimleştirilmiş Kan Tahlili PDF</option>
                      </select>
                    </div>
                    
                    <div className="border-2 border-dashed border-slate-300 rounded-xl p-8 text-center bg-slate-50 hover:bg-slate-100 transition cursor-pointer">
                      <p className="text-slate-500 font-medium">Dosyayı buraya sürükleyin veya <span className="text-teal-500 underline">göz atın</span>.</p>
                      <p className="text-xs text-slate-400 mt-2">Maksimum 50MB (.csv, .json, .pdf)</p>
                    </div>
                    
                    <div className="bg-teal-50 p-4 rounded-xl flex items-start space-x-3 text-sm text-teal-800">
                      <ShieldCheck className="w-5 h-5 shrink-0 mt-0.5 text-teal-600" />
                      <p>Yükle butonuna bastığınızda, veriniz IPFS ağına gönderilir ve erişim izni sizin cüzdan adresinize (Smart Contract üzerinden) bağlanır.</p>
                    </div>

                    <button 
                      type="submit" 
                      disabled={isProcessing}
                      className="w-full bg-teal-500 hover:bg-teal-400 text-white font-bold text-lg py-4 rounded-xl shadow-lg transition disabled:opacity-50"
                    >
                      {isProcessing ? 'Blokzincire İşleniyor...' : 'Veriyi Şifrele ve Yükle'}
                    </button>
                  </form>
                </div>
              </div>
            )}

          </div>
        )}
      </main>

      {/* --- TRANSFER MODALI (POP-UP) --- */}
      {showTransferModal && (
        <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-3xl max-w-md w-full p-8 shadow-2xl relative">
            <button 
              onClick={() => setShowTransferModal(false)}
              className="absolute top-4 right-4 text-slate-400 hover:text-slate-600 font-bold p-2"
            >
              ✕
            </button>
            <h2 className="text-2xl font-bold text-slate-800 mb-6 flex items-center">
              <Send className="w-6 h-6 mr-2 text-teal-500" />
              Token Gönder
            </h2>
            
            <form onSubmit={handleTransfer} className="space-y-5">
              <div>
                <label className="block text-sm font-medium text-slate-700 mb-1">Alıcı Cüzdan Adresi</label>
                <input 
                  type="text" 
                  value={transferTo}
                  onChange={(e) => setTransferTo(e.target.value)}
                  placeholder="0x..." 
                  className="w-full border border-slate-300 rounded-xl px-4 py-3 focus:ring-2 focus:ring-teal-500 focus:outline-none font-mono text-sm"
                  required
                />
              </div>
              
              <div>
                <div className="flex justify-between mb-1">
                  <label className="block text-sm font-medium text-slate-700">Miktar ($NEXUS)</label>
                  <span className="text-xs text-slate-500 font-medium">Bakiye: {balance.toFixed(2)}</span>
                </div>
                <div className="relative">
                  <input 
                    type="number" 
                    value={transferAmount}
                    onChange={(e) => setTransferAmount(e.target.value)}
                    placeholder="0.00" 
                    step="0.01"
                    min="0"
                    className="w-full border border-slate-300 rounded-xl px-4 py-3 pr-16 focus:ring-2 focus:ring-teal-500 focus:outline-none font-bold text-lg text-slate-800"
                    required
                  />
                  <button 
                    type="button"
                    onClick={() => setTransferAmount(balance)}
                    className="absolute right-2 top-2 bottom-2 bg-slate-100 hover:bg-slate-200 text-slate-600 text-xs font-bold px-3 rounded-lg transition"
                  >
                    MAX
                  </button>
                </div>
              </div>

              <div className="bg-slate-50 p-4 rounded-xl text-sm text-slate-600 border border-slate-100 flex justify-between">
                <span>Ağ Ücreti (Gas Fee)</span>
                <span className="font-mono text-slate-400">~0.001 NEXUS</span>
              </div>

              <button 
                type="submit" 
                disabled={isProcessing}
                className="w-full bg-slate-900 hover:bg-slate-800 text-white font-bold py-4 rounded-xl shadow-lg transition flex items-center justify-center space-x-2 disabled:opacity-50 mt-4"
              >
                {isProcessing ? (
                  <span>Ağ Onayı Bekleniyor...</span>
                ) : (
                  <>
                    <span>İşlemi Onayla</span>
                    <ArrowRight className="w-5 h-5" />
                  </>
                )}
              </button>
            </form>
          </div>
        </div>
      )}

    </div>
  );
}
