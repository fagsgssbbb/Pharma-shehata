# Pharma-shehata
صيدليه الدكتور احمد عبدالله شحاته صيدليه يوجد بها جميع الادويه 
import React, { useState, useEffect } from 'react';
import { ShoppingCart, X, LayoutDashboard, Package, Users, Store, Settings, MapPin, Phone, Plus, Bot, Menu, Trash2, LogOut, Minus, User as UserIcon } from 'lucide-react';

// =================================================================
// --- SIMULATED DATABASES & INITIAL DATA ---
// =================================================================
const initialStoreProducts = [
  { id: 1, name: 'بنادول أدفانس', price: 25.50, image: 'https://placehold.co/600x600/e0f2f1/00796b?text=Panadol', description: 'مسكن فعال وسريع للآلام.', category: 'الأدوية' },
  { id: 2, name: 'كريم مرطب سيرافي', price: 250.00, image: 'https://placehold.co/600x600/e3f2fd/1e88e5?text=CeraVe', description: 'تركيبة غنية لترطيب يدوم 24 ساعة.', category: 'العناية بالبشرة' },
];
const PHARMACY_CATEGORIES = ['الأدوية', 'العناية بالبشرة', 'العناية بالشعر', 'مستلزمات الأطفال', 'الفيتامينات والمكملات'];

// =================================================================
// --- HELPER FUNCTIONS ---
// =================================================================
const getInitialState = (key, defaultValue) => {
    try {
        const item = window.localStorage.getItem(key);
        return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
        console.error(`Error reading localStorage key “${key}”:`, error);
        return defaultValue;
    }
};

// =================================================================
// --- MAIN APP CONTROLLER ---
// =================================================================
export default function App() {
    const [view, setView] = useState('store');
    
    // All state is managed here in the root component
    const [products, setProducts] = useState(() => getInitialState('products', initialStoreProducts));
    const [customers, setCustomers] = useState(() => getInitialState('customers', []));
    const [currentUser, setCurrentUser] = useState(() => getInitialState('currentUser', null));
    const [cart, setCart] = useState([]);

    // --- Effects to save state to localStorage ---
    useEffect(() => { window.localStorage.setItem('products', JSON.stringify(products)); }, [products]);
    useEffect(() => { window.localStorage.setItem('customers', JSON.stringify(customers)); }, [customers]);
    useEffect(() => { window.localStorage.setItem('currentUser', JSON.stringify(currentUser)); }, [currentUser]);
    useEffect(() => { if (currentUser) window.localStorage.setItem(`cart_${currentUser.id}`, JSON.stringify(cart)); }, [cart, currentUser]);
    
    // --- Effect to load cart when user changes ---
    useEffect(() => {
        if (currentUser) {
            setCart(getInitialState(`cart_${currentUser.id}`, []));
        } else {
            setCart([]);
        }
    }, [currentUser]);

    // --- All handler functions are defined here and passed down ---
    const handleRegister = (userData) => { const newCustomer = { id: Date.now(), ...userData }; setCustomers(p => [...p, newCustomer]); setCurrentUser(newCustomer); };
    const handleLogin = (whatsapp, password) => { const customer = customers.find(c => c.whatsapp === whatsapp && c.password === password); if (customer) { setCurrentUser(customer); return true; } return false; };
    const handleLogout = () => setCurrentUser(null);
    const handleAddToCart = (product) => { if (!currentUser) { alert("يرجى تسجيل الدخول أولاً لإضافة منتجات للسلة."); return; } setCart(p => { const exist = p.find(i => i.id === product.id); if (exist) return p.map(i => i.id === product.id ? { ...i, quantity: i.quantity + 1 } : i); return [...p, { ...product, quantity: 1 }]; }); };
    const handleRemoveFromCart = (productId) => setCart(p => p.filter(i => i.id !== productId));
    const updateCartQuantity = (productId, quantity) => { if (quantity < 1) { handleRemoveFromCart(productId); return; } setCart(p => p.map(i => i.id === productId ? { ...i, quantity } : i)); };

    const systemProps = {
        products, setProducts,
        customers, setCustomers,
        currentUser, onRegister: handleRegister, onLogin: handleLogin, onLogout: handleLogout,
        cart, onAddToCart: handleAddToCart, onRemoveFromCart: handleRemoveFromCart, onUpdateCartQuantity: updateCartQuantity
    };

    return (
        <div dir="rtl">
            <div className="bg-gray-900 text-white p-2">
                <div className="container mx-auto flex justify-center items-center space-x-reverse space-x-4">
                    <span className="font-semibold">تبديل التطبيق:</span>
                    <button onClick={() => setView('store')} className={`flex items-center px-4 py-2 rounded-md transition ${view === 'store' ? 'bg-blue-600' : 'bg-gray-700 hover:bg-gray-600'}`}><Store className="ml-2" /> واجهة المتجر</button>
                    <button onClick={() => setView('admin')} className={`flex items-center px-4 py-2 rounded-md transition ${view === 'admin' ? 'bg-blue-600' : 'bg-gray-700 hover:bg-gray-600'}`}><Settings className="ml-2" /> لوحة التحكم</button>
                </div>
            </div>
            {view === 'store' ? <Storefront {...systemProps} /> : <AdminDashboard {...systemProps} />}
        </div>
    );
}

// =================================================================
// --- STOREFRONT COMPONENTS ---
// =================================================================
const Storefront = ({ products, currentUser, onRegister, onLogin, onLogout, cart, onAddToCart, onRemoveFromCart, onUpdateCartQuantity }) => {
    const [showWelcome, setShowWelcome] = useState(() => !sessionStorage.getItem('welcomeShown'));
    const [showAuth, setShowAuth] = useState(false);
    const [isCartOpen, setIsCartOpen] = useState(false);
    const [isNavOpen, setIsNavOpen] = useState(false);
    const [filteredCategory, setFilteredCategory] = useState(null);
    const handleStart = () => { sessionStorage.setItem('welcomeShown', 'true'); setShowWelcome(false); };
    if (showWelcome) return <WelcomeScreen onStart={handleStart} />;
    const displayedProducts = filteredCategory ? products.filter(p => p.category === filteredCategory) : products;
    return (
        <div className="bg-gray-100 min-h-screen">
            <header className="bg-white shadow-sm sticky top-0 z-40"><div className="container mx-auto px-4 py-3 flex justify-between items-center"><button onClick={() => setIsNavOpen(true)} className="text-gray-600"><Menu className="h-7 w-7" /></button><a href="#" onClick={() => setFilteredCategory(null)} className="text-xl md:text-2xl font-extrabold text-gray-800">صيدلية د. احمد شحاته</a><div className="flex items-center space-x-3"><button onClick={() => setIsCartOpen(true)} className="relative text-gray-600"><ShoppingCart className="h-7 w-7" />{cart.length > 0 && <span className="absolute -top-2 -right-2 bg-blue-500 text-white text-xs rounded-full h-5 w-5 flex items-center justify-center">{cart.reduce((s, i) => s + i.quantity, 0)}</span>}</button><ProfileIcon currentUser={currentUser} onLoginClick={() => setShowAuth(true)} onLogout={onLogout} /></div></div></header>
            <main className="container mx-auto px-4 py-8"><h2 className="text-2xl font-bold text-gray-800 my-8">{filteredCategory || 'كل المنتجات'}</h2><div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4 md:gap-6">{displayedProducts.map(p => <ProductCard key={p.id} product={p} onAddToCart={onAddToCart} />)}</div></main>
            <Footer /><CartSidebar isOpen={isCartOpen} onClose={() => setIsCartOpen(false)} cart={cart} onRemove={onRemoveFromCart} onUpdateQuantity={onUpdateCartQuantity} /><NavSidebar isOpen={isNavOpen} onClose={() => setIsNavOpen(false)} onSelectCategory={setFilteredCategory} />
            {showAuth && !currentUser && <AuthScreen onRegister={onRegister} onLogin={onLogin} onClose={() => setShowAuth(false)} />}
        </div>
    );
};
const WelcomeScreen = ({ onStart }) => (<div className="fixed inset-0 bg-gray-900 z-50 flex items-center justify-center p-4 bg-cover bg-center" style={{backgroundImage: "url('https://placehold.co/1920x1080/004d40/004d40?text=.')"}}><div className="text-center text-white"><h1 className="text-3xl md:text-5xl font-bold">أهلاً بكم في صيدلية</h1><h2 className="text-4xl md:text-6xl font-extrabold text-blue-300 my-4">د. احمد عبدالله شحاته</h2><p className="text-lg md:text-xl mb-8">تشرفنا بزيارتكم</p><button onClick={onStart} className="bg-white text-blue-600 font-bold py-3 px-10 rounded-full text-lg hover:bg-blue-100 transition-transform transform hover:scale-105">فل نبدأ</button></div></div>);
const AuthScreen = ({ onRegister, onLogin, onClose }) => { const [isLoginView, setIsLoginView] = useState(true); const [error, setError] = useState(''); const [formData, setFormData] = useState({ fullName: '', address: '', whatsapp: '', password: '', whatsappConfirm: '' }); const handleChange = (e) => setFormData({...formData, [e.target.name]: e.target.value }); const handleRegisterSubmit = (e) => { e.preventDefault(); if (formData.whatsapp !== formData.whatsappConfirm) { setError('رقما الواتساب غير متطابقين.'); return; } if (formData.password.length < 8) { setError('كلمة السر يجب أن تكون 8 أحرف على الأقل.'); return; } setError(''); onRegister(formData); onClose(); }; const handleLoginSubmit = (e) => { e.preventDefault(); const success = onLogin(formData.whatsapp, formData.password); if (!success) setError('رقم الواتساب أو كلمة السر غير صحيحة.'); else onClose(); }; return (<div className="fixed inset-0 bg-black bg-opacity-60 z-50 flex items-center justify-center p-4" onClick={onClose}><div className="bg-white rounded-2xl shadow-2xl w-full max-w-md p-6 md:p-8 relative" onClick={e => e.stopPropagation()}><button onClick={onClose} className="absolute top-4 left-4 text-gray-400 hover:text-gray-600"><X /></button><div className="flex border-b mb-6"><button onClick={() => setIsLoginView(true)} className={`flex-1 py-3 font-bold ${isLoginView ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-500'}`}>تسجيل الدخول</button><button onClick={() => setIsLoginView(false)} className={`flex-1 py-3 font-bold ${!isLoginView ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-500'}`}>حساب جديد</button></div>{error && <p className="text-red-500 text-sm text-center mb-4">{error}</p>}{isLoginView ? (<form onSubmit={handleLoginSubmit} className="space-y-4"><input name="whatsapp" type="tel" placeholder="رقم الواتساب" onChange={handleChange} required className="w-full p-3 border rounded-lg" /><input name="password" type="password" placeholder="كلمة السر" onChange={handleChange} required className="w-full p-3 border rounded-lg" /><button type="submit" className="w-full bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700">دخول</button></form>) : (<form onSubmit={handleRegisterSubmit} className="space-y-4"><input name="fullName" type="text" placeholder="الاسم الرباعي" onChange={handleChange} required className="w-full p-3 border rounded-lg" /><textarea name="address" placeholder="العنوان بالتفصيل" onChange={handleChange} required className="w-full p-3 border rounded-lg h-20"></textarea><input name="whatsapp" type="tel" placeholder="رقم الواتساب" onChange={handleChange} required className="w-full p-3 border rounded-lg" /><input name="whatsappConfirm" type="tel" placeholder="تأكيد رقم الواتساب" onChange={handleChange} required className="w-full p-3 border rounded-lg" /><input name="password" type="password" placeholder="كلمة السر (8 أحرف على الأقل)" onChange={handleChange} required className="w-full p-3 border rounded-lg" /><button type="submit" className="w-full bg-green-600 text-white font-bold py-3 rounded-lg hover:bg-green-700">إنشاء حساب</button></form>)}</div></div>); };
const ProfileIcon = ({ currentUser, onLoginClick, onLogout }) => { const [isOpen, setIsOpen] = useState(false); if (!currentUser) return <button onClick={onLoginClick} className="bg-blue-500 text-white px-4 py-2 rounded-full text-sm font-semibold">تسجيل الدخول</button>; return (<div className="relative"><button onClick={() => setIsOpen(!isOpen)} className="w-10 h-10 bg-gray-200 rounded-full flex items-center justify-center"><UserIcon className="text-gray-600" /></button>{isOpen && <div className="absolute left-0 mt-2 w-64 bg-white rounded-lg shadow-xl z-50 p-4"><p className="font-bold">{currentUser.fullName}</p><p className="text-sm text-gray-500 mb-4">{currentUser.whatsapp}</p><button onClick={onLogout} className="w-full text-left flex items-center p-2 rounded-lg hover:bg-gray-100 text-red-500"><LogOut className="ml-2" size={16} /> تسجيل الخروج</button></div>}</div>); };
const ProductCard = ({ product, onAddToCart }) => (<div className="bg-white rounded-lg shadow-md overflow-hidden group transition-all duration-300 hover:shadow-2xl hover:-translate-y-1"><img src={product.image} alt={product.name} className="w-full h-40 md:h-48 object-cover" /><div className="p-3 md:p-4"><h3 className="text-sm md:text-md font-semibold text-gray-800 truncate h-10">{product.name}</h3><div className="flex justify-between items-center mt-2"><p className="text-md md:text-lg font-bold text-blue-600">{product.price.toFixed(2)} ج.م</p><button onClick={() => onAddToCart(product)} className="bg-blue-100 text-blue-600 p-2 rounded-full hover:bg-blue-200 transition-colors"><Plus size={16}/></button></div></div></div>);
const CartSidebar = ({ isOpen, onClose, cart, onRemove, onUpdateQuantity }) => { const subtotal = cart.reduce((s, i) => s + (i.price * i.quantity), 0); return (<div className={`fixed inset-0 z-50 transition-opacity duration-300 ${isOpen ? 'bg-black bg-opacity-50' : 'opacity-0 pointer-events-none'}`} onClick={onClose}><div className={`absolute top-0 right-0 h-full w-full max-w-sm bg-white shadow-xl transform transition-transform duration-300 ${isOpen ? 'translate-x-0' : 'translate-x-full'}`} onClick={e => e.stopPropagation()}><div className="flex flex-col h-full"><div className="flex justify-between items-center p-4 border-b"><h2 className="text-xl font-bold">عربة التسوق</h2><button onClick={onClose}><X /></button></div><div className="flex-1 p-4 overflow-y-auto">{cart.length === 0 ? <p className="text-center text-gray-500 mt-8">عربة التسوق فارغة.</p> : cart.map(item => (<div key={item.id} className="flex items-center mb-4"><img src={item.image} className="w-16 h-16 rounded-md object-cover mr-4" /><div className="flex-1"><p className="font-semibold">{item.name}</p><p className="text-sm text-gray-500">{item.quantity} x {item.price.toFixed(2)} ج.م</p></div><div className="flex items-center"><button onClick={() => onUpdateQuantity(item.id, item.quantity + 1)} className="p-1 border rounded-full"><Plus size={12}/></button><span className="px-3">{item.quantity}</span><button onClick={() => onUpdateQuantity(item.id, item.quantity - 1)} className="p-1 border rounded-full"><Minus size={12}/></button></div><button onClick={() => onRemove(item.id)} className="text-red-500 mr-3"><Trash2 size={16} /></button></div>))}</div><div className="p-4 border-t"><div className="flex justify-between font-bold text-lg mb-4"><span>المجموع</span><span>{subtotal.toFixed(2)} ج.م</span></div><button className="w-full bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700">إتمام الشراء</button></div></div></div></div>); };
const NavSidebar = ({ isOpen, onClose, onSelectCategory }) => (<div className={`fixed inset-0 z-50 transition-opacity duration-300 ${isOpen ? 'bg-black bg-opacity-50' : 'opacity-0 pointer-events-none'}`} onClick={onClose}><div className={`absolute top-0 right-0 h-full w-full max-w-sm bg-white shadow-xl transform transition-transform duration-300 ${isOpen ? 'translate-x-0' : 'translate-x-full'}`} onClick={e => e.stopPropagation()}><div className="flex justify-between items-center p-4 border-b"><h2 className="text-xl font-bold">القائمة</h2><button onClick={onClose}><X /></button></div><div className="p-4"><nav className="space-y-2"><h3 className="font-bold text-gray-500 text-sm px-3 mt-4 mb-2">الأقسام</h3><a href="#" onClick={() => { onSelectCategory(null); onClose(); }} className="block p-3 rounded-lg hover:bg-gray-100">كل المنتجات</a>{PHARMACY_CATEGORIES.map(cat => <a key={cat} href="#" onClick={() => { onSelectCategory(cat); onClose(); }} className="block p-3 rounded-lg hover:bg-gray-100">{cat}</a>)}</nav></div></div></div>);
const HeroCarousel = () => (<div className="relative w-full h-56 md:h-72 overflow-hidden rounded-lg shadow-lg"><img src="https://placehold.co/1200x500/004d40/ffffff?text=عروض+خاصة+لفترة+محدودة" className="w-full h-full object-cover" /></div>);
const Footer = () => (<footer className="bg-gray-800 text-white mt-12 py-8"><div className="container mx-auto text-center"><p>صيدلية د. احمد عبدالله شحاته</p><p className="text-sm text-gray-400">العنوان: كفر حمد موسي - ابوكبير - شرقيه | للتواصل: 01093239306 - 01147676867</p></div></footer>);

// =================================================================
// --- ADMIN DASHBOARD COMPONENTS ---
// =================================================================
const AdminDashboard = ({ customers, setCustomers, products, setProducts }) => {
    const [view, setView] = useState('dashboard');
    const [isSidebarOpen, setIsSidebarOpen] = useState(false);
    const handleSetView = (v) => { setView(v); setIsSidebarOpen(false); };
    return (<div className="flex h-screen bg-gray-100 overflow-hidden"><AdminSidebar view={view} setView={handleSetView} isSidebarOpen={isSidebarOpen} setIsSidebarOpen={setIsSidebarOpen} /><div className="flex-1 flex flex-col"><header className="bg-white shadow-sm p-4 flex justify-between items-center z-10"><button onClick={() => setIsSidebarOpen(true)} className="md:hidden text-gray-600"><Menu /></button><div></div></header><main className="flex-1 p-4 md:p-8 overflow-y-auto"><AdminContent view={view} customers={customers} setCustomers={setCustomers} products={products} setProducts={setProducts} /></main></div></div>);
};
const AdminSidebar = ({ view, setView, isSidebarOpen, setIsSidebarOpen }) => (<aside className={`fixed top-0 right-0 h-full bg-gray-800 text-white flex flex-col w-64 z-30 transform transition-transform duration-300 ease-in-out ${isSidebarOpen ? 'translate-x-0' : 'translate-x-full'} md:relative md:translate-x-0`}><div className="p-4 text-2xl font-bold border-b border-gray-700 flex justify-between items-center"><span>لوحة التحكم</span><button onClick={() => setIsSidebarOpen(false)} className="md:hidden text-white"><X /></button></div><nav className="flex-1 p-2 space-y-2">{[{v: 'dashboard', t: 'لوحة المعلومات', i: LayoutDashboard}, {v: 'products', t: 'إدارة المنتجات', i: Package}, {v: 'drug_ai', t: 'إضافة عبر Drug AI', i: Bot}, {v: 'customers', t: 'إدارة العملاء', i: Users}].map(item => (<a key={item.v} href="#" onClick={() => setView(item.v)} className={`flex items-center p-3 rounded-lg hover:bg-gray-700 ${view === item.v ? 'bg-blue-600' : ''}`}><item.i className="ml-3" /> {item.t}</a>))}</nav></aside>);
const AdminContent = ({ view, customers, setCustomers, products, setProducts }) => {
    const [productForm, setProductForm] = useState({ name: '', price: '', category: PHARMACY_CATEGORIES[0], image: null, imagePreview: '', description: '' });
    const [dbSearchTerm, setDbSearchTerm] = useState('');
    const [dbSearchResults, setDbSearchResults] = useState([]);
    const handleProductFormChange = (e) => setProductForm({...productForm, [e.target.name]: e.target.value });
    const handleProductImageChange = (e) => { if(e.target.files[0]) setProductForm({...productForm, image: e.target.files[0], imagePreview: URL.createObjectURL(e.target.files[0])})};
    const handleAddProduct = (e) => { e.preventDefault(); setProducts(p => [{ id: Date.now(), ...productForm, image: productForm.imagePreview }, ...p]); setProductForm({ name: '', price: '', category: PHARMACY_CATEGORIES[0], image: null, imagePreview: '', description: '' }); };
    const handleSearchInDB = () => setDbSearchResults(DRUG_AI_DATABASE.filter(p => p.name.includes(dbSearchTerm)));
 
