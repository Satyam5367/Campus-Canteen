
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Campus Canteen</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body class="bg-gray-100 transition duration-300" id="body">
  <div class="p-4 flex justify-between items-center">
    <h1 class="text-3xl font-bold">Campus Canteen</h1>
    <button onclick="toggleDarkMode()" class="bg-gray-300 px-3 py-1 rounded">🌓 Toggle Dark</button>
  </div>

  <div id="root" class="p-4"></div>
  <div id="toast-container" class="fixed top-5 right-5 space-y-2 z-50"></div>

  <script type="text/babel">

    const { useState, useEffect } = React;

    const menu = [
      { id: 1, name: "Veg Sandwich", desc: "Grilled veggies", price: 50, image: "https://via.placeholder.com/150" },
      { id: 2, name: "Paneer Masala", desc: "Creamy curry", price: 120, image: "https://via.placeholder.com/150" },
      { id: 3, name: "Masala Chai", desc: "Spiced tea", price: 20, image: "https://via.placeholder.com/150" }
    ];

    function App() {
      const [cart, setCart] = useState([]);
      const [orders, setOrders] = useState([]);
      const [spices, setSpices] = useState({});
      const [viewOrders, setViewOrders] = useState(false);
      const [favorites, setFavorites] = useState([]);

      useEffect(() => {
        const savedOrders = JSON.parse(localStorage.getItem("orders") || "[]");
        const savedFavs = JSON.parse(localStorage.getItem("favorites") || "[]");
        setOrders(savedOrders);
        setFavorites(savedFavs);
      }, []);

      useEffect(() => {
        const interval = setInterval(() => {
          const updated = orders.map(order =>
            order.status === "Preparing"
              ? { ...order, status: "Ready" }
              : order
          );
          setOrders(updated);
          localStorage.setItem("orders", JSON.stringify(updated));
        }, 15000);
        return () => clearInterval(interval);
      }, [orders]);

      const toast = (msg) => {
        const toastEl = document.createElement('div');
        toastEl.className = "bg-green-500 text-white px-4 py-2 rounded shadow";
        toastEl.innerText = msg;
        document.getElementById('toast-container').appendChild(toastEl);
        setTimeout(() => toastEl.remove(), 3000);
      };

      const toggleFavorite = (id) => {
        const updated = favorites.includes(id)
          ? favorites.filter(f => f !== id)
          : [...favorites, id];
        setFavorites(updated);
        localStorage.setItem("favorites", JSON.stringify(updated));
      };

      const handleSpiceChange = (id, value) => {
        setSpices(prev => ({ ...prev, [id]: value }));
      };

      const addToCart = (item) => {
        const spice = spices[item.id] || "Medium";
        const key = `${item.id}-${spice}`;
        setCart(prev => {
          const found = prev.find(i => i.key === key);
          return found
            ? prev.map(i => i.key === key ? { ...i, qty: i.qty + 1 } : i)
            : [...prev, { ...item, qty: 1, spice, key }];
        });
        toast(`${item.name} added!`);
      };

      const changeQty = (key, delta) => {
        setCart(prev => prev.map(i => i.key === key ? { ...i, qty: Math.max(1, i.qty + delta) } : i));
      };

      const removeFromCart = (key) => {
        setCart(cart.filter(i => i.key !== key));
      };

      const placeOrder = () => {
        const newOrder = {
          items: cart,
          total: cart.reduce((sum, i) => sum + i.price * i.qty, 0),
          status: "Preparing",
          time: new Date().toLocaleTimeString()
        };
        const updated = [...orders, newOrder];
        localStorage.setItem("orders", JSON.stringify(updated));
        setOrders(updated);
        setCart([]);
        toast("Order placed!");
      };

      const clearOrders = () => {
        localStorage.removeItem("orders");
        setOrders([]);
        toast("All orders cleared");
      };

      return (
        <div className="max-w-4xl mx-auto">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-2xl font-semibold">{viewOrders ? "My Orders" : "Menu"}</h2>
            <button onClick={() => setViewOrders(!viewOrders)} className="bg-blue-500 text-white px-3 py-1 rounded">
              {viewOrders ? "← Back to Menu" : "📦 My Orders"}
            </button>
          </div>

          {viewOrders ? (
            <div>
              {orders.length === 0 ? <p>No orders yet.</p> :
                orders.map((order, i) => (
                  <div key={i} className="bg-white p-4 rounded-xl shadow mb-4">
                    <p className="font-bold mb-2">Order #{i + 1} – <span className="text-sm italic">{order.status}</span></p>
                    <ul className="text-sm mb-2">
                      {order.items.map((item, j) => (
                        <li key={j}>{item.name} × {item.qty} ({item.spice})</li>
                      ))}
                    </ul>
                    <p className="text-sm">Total: ₹{order.total} at {order.time}</p>
                  </div>
                ))}
              <button onClick={clearOrders} className="bg-red-500 text-white px-4 py-1 rounded">
                Clear All Orders
              </button>
            </div>
          ) : (
            <>
              <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-4 mb-6">
                {menu.map(item => (
                  <div key={item.id} className="bg-white p-4 rounded-xl shadow relative">
                    <button
                      className="absolute top-2 right-2 text-xl"
                      onClick={() => toggleFavorite(item.id)}
                    >
                      {favorites.includes(item.id) ? "⭐" : "☆"}
                    </button>
                    <img src={item.image} alt={item.name} className="mb-2 rounded" />
                    <h3 className="font-bold">{item.name}</h3>
                    <p className="text-sm text-gray-500">{item.desc}</p>
                    <p className="mt-1 font-semibold">₹{item.price}</p>
                    <select onChange={(e) => handleSpiceChange(item.id, e.target.value)} className="mt-2 w-full border p-1">
                      <option value="Mild">Mild</option>
                      <option value="Medium" selected>Medium</option>
                      <option value="Spicy">Spicy</option>
                    </select>
                    <button onClick={() => addToCart(item)} className="mt-2 w-full bg-green-500 text-white px-3 py-1 rounded">
                      Add to Cart
                    </button>
                  </div>
                ))}
              </div>

              {cart.length > 0 && (
                <div className="bg-white p-4 rounded-xl shadow mb-6">
                  <h2 className="text-xl font-bold mb-2">Cart</h2>
                  {cart.map(item => (
                    <div key={item.key} className="flex justify-between items-center mb-2">
                      <span>{item.name} ({item.spice}) × {item.qty}</span>
                      <div className="flex gap-1">
                        <button onClick={() => changeQty(item.key, -1)} className="bg-gray-200 px-2">–</button>
                        <button onClick={() => changeQty(item.key, 1)} className="bg-gray-200 px-2">+</button>
                        <button onClick={() => removeFromCart(item.key)} className="text-red-500">×</button>
                      </div>
                    </div>
                  ))}
                  <p className="font-bold mt-2">
                    Total: ₹{cart.reduce((sum, i) => sum + i.price * i.qty, 0)}
                  </p>
                  <button onClick={placeOrder} className="mt-3 w-full bg-blue-600 text-white px-4 py-2 rounded">
                    Place Order
                  </button>
                </div>
              )}
            </>
          )}
        </div>
      );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<App />);

    function toggleDarkMode() {
      const body = document.getElementById('body');
      body.classList.toggle('bg-gray-900');
      body.classList.toggle('text-white');
    }
  </script>
</body>
</html>
