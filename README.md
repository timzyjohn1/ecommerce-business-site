const products = [
  {
    id: 1,
    name: "Cloud Lounge Chair",
    category: "home",
    price: 229,
    rating: 4.9,
    badge: "Bestseller",
    image:
      "https://images.unsplash.com/photo-1505693416388-ac5ce068fe85?auto=format&fit=crop&w=900&q=80"
  },
  {
    id: 2,
    name: "Fjord Ceramic Vase",
    category: "home",
    price: 58,
    rating: 4.8,
    badge: "New",
    image:
      "https://images.unsplash.com/photo-1494438639946-1ebd1d20bf85?auto=format&fit=crop&w=900&q=80"
  },
  {
    id: 3,
    name: "Aero Smart Lamp",
    category: "tech",
    price: 124,
    rating: 4.9,
    badge: "Popular",
    image:
      "https://images.unsplash.com/photo-1515377905703-c4788e51af15?auto=format&fit=crop&w=900&q=80"
  },
  {
    id: 4,
    name: "Arc Journal Set",
    category: "accessories",
    price: 32,
    rating: 4.7,
    badge: "Giftable",
    image:
      "https://images.unsplash.com/photo-1455390582262-044cdead277a?auto=format&fit=crop&w=900&q=80"
  },
  {
    id: 5,
    name: "Halo Diffuser",
    category: "wellness",
    price: 68,
    rating: 4.8,
    badge: "Wellness",
    image:
      "https://images.unsplash.com/photo-1528740561666-dc2479dc08ab?auto=format&fit=crop&w=900&q=80"
  },
  {
    id: 6,
    name: "Aura Wireless Speaker",
    category: "tech",
    price: 149,
    rating: 4.9,
    badge: "Top rated",
    image:
      "https://images.unsplash.com/photo-1546435770-a3e426bf472b?auto=format&fit=crop&w=900&q=80"
  },
  {
    id: 7,
    name: "Luma Throw Blanket",
    category: "home",
    price: 89,
    rating: 4.8,
    badge: "Cozy",
    image:
      "https://images.unsplash.com/photo-1517705008128-361805f42e86?auto=format&fit=crop&w=900&q=80"
  },
  {
    id: 8,
    name: "Bloom Tea Kit",
    category: "wellness",
    price: 44,
    rating: 4.7,
    badge: "Limited",
    image:
      "https://images.unsplash.com/photo-1515823064-d6e0c04616a7?auto=format&fit=crop&w=900&q=80"
  }
];

const productGrid = document.getElementById("product-grid");
const filterButtons = document.querySelectorAll(".filter-btn");
const cartItems = document.getElementById("cart-items");
const cartCount = document.querySelector(".cart-count");
const cartPanel = document.querySelector(".cart-panel");
const cartOverlay = document.querySelector(".cart-overlay");
const subtotalEl = document.getElementById("subtotal");
const yearEl = document.getElementById("year");

let activeFilter = "all";
let cart = [];

const formatCurrency = (value) =>
  new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" }).format(value);

function renderProducts() {
  const visibleProducts =
    activeFilter === "all"
      ? products
      : products.filter((product) => product.category === activeFilter);

  productGrid.innerHTML = visibleProducts
    .map(
      (product) => `
        <article class="product-card">
          <div class="product-image-wrap">
            <img src="${product.image}" alt="${product.name}" />
            <span class="product-badge">${product.badge}</span>
          </div>
          <div class="product-info">
            <div class="product-rating">★★★★★ <span>${product.rating}</span></div>
            <h3 class="product-name">${product.name}</h3>
            <div class="product-meta">
              <span class="product-price">${formatCurrency(product.price)}</span>
              <button class="add-to-cart" type="button" data-id="${product.id}" aria-label="Add ${product.name} to cart">+</button>
            </div>
          </div>
        </article>
      `
    )
    .join("");
}

function updateCart() {
  const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
  const subtotal = cart.reduce((sum, item) => sum + item.quantity * item.price, 0);

  cartCount.textContent = totalItems;
  subtotalEl.textContent = formatCurrency(subtotal);

  if (!cart.length) {
    cartItems.innerHTML = '<p class="empty-cart">Your cart is empty.</p>';
    return;
  }

  cartItems.innerHTML = cart
    .map(
      (item) => `
        <div class="cart-item">
          <img src="${item.image}" alt="${item.name}" />
          <div class="cart-item-info">
            <strong>${item.name}</strong>
            <span>${formatCurrency(item.price)}</span>
            <div class="cart-qty">
              <button type="button" class="qty-btn" data-action="decrease" data-id="${item.id}" aria-label="Decrease quantity">−</button>
              <span>${item.quantity}</span>
              <button type="button" class="qty-btn" data-action="increase" data-id="${item.id}" aria-label="Increase quantity">+</button>
            </div>
          </div>
          <div class="cart-item-price">${formatCurrency(item.quantity * item.price)}</div>
        </div>
      `
    )
    .join("");
}

function addToCart(productId) {
  const product = products.find((item) => item.id === Number(productId));
  if (!product) return;

  const existingItem = cart.find((item) => item.id === product.id);

  if (existingItem) {
    existingItem.quantity += 1;
  } else {
    cart.push({ ...product, quantity: 1 });
  }

  updateCart();
  openCart();
}

function changeQuantity(productId, action) {
  const item = cart.find((entry) => entry.id === Number(productId));
  if (!item) return;

  if (action === "increase") item.quantity += 1;
  if (action === "decrease") item.quantity -= 1;

  if (item.quantity <= 0) {
    cart = cart.filter((entry) => entry.id !== Number(productId));
  }

  updateCart();
}

function openCart() {
  cartPanel.classList.add("open");
  cartOverlay.classList.add("visible");
}

function closeCart() {
  cartPanel.classList.remove("open");
  cartOverlay.classList.remove("visible");
}

filterButtons.forEach((button) => {
  button.addEventListener("click", () => {
    activeFilter = button.dataset.filter;
    filterButtons.forEach((btn) => btn.classList.toggle("active", btn === button));
    renderProducts();
  });
});

document.addEventListener("click", (event) => {
  const addButton = event.target.closest(".add-to-cart");
  if (addButton) {
    addToCart(addButton.dataset.id);
  }

  const qtyButton = event.target.closest(".qty-btn");
  if (qtyButton) {
    changeQuantity(qtyButton.dataset.id, qtyButton.dataset.action);
  }

  if (event.target.matches(".cart-button")) {
    openCart();
  }

  if (event.target.matches(".close-cart") || event.target.matches(".cart-overlay")) {
    closeCart();
  }
});

document.querySelector(".newsletter-form").addEventListener("submit", (event) => {
  event.preventDefault();
  const input = event.target.querySelector("input");
  input.value = "";
  input.placeholder = "Thanks for joining!";
});

yearEl.textContent = new Date().getFullYear();

renderProducts();
updateCart();
