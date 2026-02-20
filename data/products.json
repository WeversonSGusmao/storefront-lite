/* Storefront Lite — JS puro, acessível e performático */
const $ = sel => document.querySelector(sel);
const $$ = sel => document.querySelectorAll(sel);

const state = {
  products: [],
  filtered: [],
  cart: JSON.parse(localStorage.getItem('cart') || '[]'),
  query: '',
  category: 'all',
  sort: 'name-asc'
};

const fmtBRL = v => v.toLocaleString('pt-BR', { style:'currency', currency:'BRL' });

async function loadProducts() {
  try {
    const res = await fetch('data/products.json', { cache: 'no-store' });
    state.products = await res.json();
    initFilters();
    applyFilters();
  } catch (e) {
    console.error('Erro ao carregar produtos', e);
  } finally {
    $('section[aria-live]').setAttribute('aria-busy','false');
  }
}

/* Filtros / Ordenação */
function initFilters() {
  const cats = ['all', ...new Set(state.products.map(p => p.category))];
  const sel = $('#category');
  cats.forEach(c => {
    const o = document.createElement('option');
    o.value = c; o.textContent = c === 'all' ? 'Todas' : c;
    sel.appendChild(o);
  });
}

function applyFilters() {
  const q = state.query.trim().toLowerCase();
  let list = state.products.filter(p =>
    (state.category === 'all' || p.category === state.category) &&
    (p.name.toLowerCase().includes(q) || p.description.toLowerCase().includes(q))
  );

  const [key, dir] = state.sort.split('-'); // name|price + asc|desc
  list.sort((a,b) => {
    const A = key === 'price' ? a.price : a.name.localeCompare ? a.name : a.name.toString();
    const B = key === 'price' ? b.price : b.name.localeCompare ? b.name : b.name.toString();
    const cmp = key === 'price' ? (A - B) : a.name.localeCompare(b.name);
    return dir === 'asc' ? cmp : -cmp;
  });

  state.filtered = list;
  renderGrid();
}

/* Renderização */
function el(tag, cls, html) { const e = document.createElement(tag); if (cls) e.className = cls; if (html) e.innerHTML = html; return e; }

function renderGrid() {
  const grid = $('#grid');
  grid.innerHTML = '';
  $('#empty').hidden = state.filtered.length > 0;

  for (const p of state.filtered) {
    const li = el('li', 'card');

    // imagem placeholder (evita dependência de assets)
    const img = el('div', 'card__img', p.emoji ?? '📦');
    img.setAttribute('role','img');
    img.setAttribute('aria-label', p.name);

    const body = el('div','card__body');
    const h3 = el('h3','card__title', p.name);
    const desc = el('p', '', p.description);
    const price = el('p','card__price', fmtBRL(p.price));
    const actions = el('div','card__actions');
    const btn = el('button','btn primary','Adicionar');

    btn.addEventListener('click', () => addToCart(p));

    actions.appendChild(btn);
    body.append(h3, desc, price, actions);
    li.append(img, body);
    grid.appendChild(li);
  }
}

/* Carrinho */
function addToCart(product) {
  const idx = state.cart.findIndex(i => i.id === product.id);
  if (idx >= 0) state.cart[idx].qty += 1;
  else state.cart.push({ id: product.id, name: product.name, price: product.price, qty: 1 });
  persistCart(); renderCart();
}

function removeFromCart(id) {
  const idx = state.cart.findIndex(i => i.id === id);
  if (idx >= 0) {
    state.cart[idx].qty -= 1;
    if (state.cart[idx].qty <= 0) state.cart.splice(idx,1);
  }
  persistCart(); renderCart();
}

function clearCart() { state.cart = []; persistCart(); renderCart(); }
function cartSubtotal() { return state.cart.reduce((s,i)=> s + i.price * i.qty, 0); }
function persistCart() { localStorage.setItem('cart', JSON.stringify(state.cart)); }

function renderCart() {
  $('#cart-count').textContent = state.cart.reduce((s,i)=>s+i.qty,0);
  $('#cart-subtotal').textContent = fmtBRL(cartSubtotal());
  const ul = $('#cart-items'); ul.innerHTML = '';
  for (const item of state.cart) {
    const li = el('li','cart__item');
    li.append(
      el('div','', item.qty+'×'),
      el('div','', `<strong>${item.name}</strong><br><small>${fmtBRL(item.price)}</small>`),
      (()=>{ const g = el('div'); 
        const minus = el('button','icon-btn','−'); minus.setAttribute('aria-label','Remover uma unidade'); 
        const plus = el('button','icon-btn','+'); plus.setAttribute('aria-label','Adicionar uma unidade');
        minus.addEventListener('click', ()=> removeFromCart(item.id));
        plus.addEventListener('click', ()=> addToCart(item));
        g.append(minus, plus); return g; })()
    );
    ul.appendChild(li);
  }
}

/* Tema (dark/light) com persistência */
(function themeInit(){
  const root = document.documentElement;
  const key = 'theme';
  const btn = $('#theme-toggle'); const icon = $('#theme-icon');

  function setMeta(color){ let m = document.querySelector('meta[name="theme-color"]');
    if(!m){ m = document.createElement('meta'); m.setAttribute('name','theme-color'); document.head.appendChild(m); }
    m.setAttribute('content', color);
  }

  function apply(theme, persist=true){
    root.classList.remove('force-light','force-dark');
    if(theme === 'dark'){ root.classList.add('dark','force-dark'); icon.textContent='🌞'; if (persist) localStorage.setItem(key,'dark'); setMeta('#0b1220'); }
    else if(theme === 'light'){ root.classList.remove('dark'); root.classList.add('force-light'); icon.textContent='🌙'; if (persist) localStorage.setItem(key,'light'); setMeta('#ffffff'); }
    else { // seguir sistema
      localStorage.removeItem(key);
      if (matchMedia('(prefers-color-scheme: dark)').matches) { root.classList.add('dark'); icon.textContent='🌞'; setMeta('#0b1220'); }
      else { icon.textContent='🌙'; setMeta('#ffffff'); }
    }
  }

  const saved = localStorage.getItem(key);
  apply(saved ?? null, false);
  matchMedia('(prefers-color-scheme: dark)').addEventListener('change', ()=> { if(!localStorage.getItem(key)) apply(null,false); });
  btn?.addEventListener('click', ()=>{
    const isDark = root.classList.contains('dark');
    apply(isDark ? 'light' : 'dark', true);
  });
})();

/* UI do carrinho (painel) */
(function cartPanelInit() {
  const toggle = $('#cart-toggle'), panel = $('#cart'), close = $('#cart-close');
  toggle.addEventListener('click', ()=>{
    const open = toggle.getAttribute('aria-expanded') === 'true';
    toggle.setAttribute('aria-expanded', String(!open));
    if (open) { panel.setAttribute('hidden',''); toggle.focus(); }
    else { panel.removeAttribute('hidden'); panel.querySelector('h2')?.focus(); }
  });
  close.addEventListener('click', ()=> { toggle.click(); });
  $('#cart-clear').addEventListener('click', clearCart);
  $('#checkout').addEventListener('click', ()=> alert('Fluxo de checkout demonstrativo.'));
})();

/* Inputs: busca, categoria, ordenação */
(function controlsInit(){
  $('#q').addEventListener('input', (e)=> { state.query = e.target.value; applyFilters(); });
  $('#category').addEventListener('change', (e)=> { state.category = e.target.value; applyFilters(); });
  $('#sort').addEventListener('change', (e)=> { state.sort = e.target.value; applyFilters(); });
})();

/* Inicialização */
renderCart();
loadProducts();
