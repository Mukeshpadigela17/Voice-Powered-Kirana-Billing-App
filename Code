import os, textwrap, zipfile, json, pathlib

base = pathlib.Path("/mnt/data/kirana-billing-app")
(base / "static").mkdir(parents=True, exist_ok=True)

files = {
"app.py": r'''from datetime import datetime, date
from io import BytesIO
import csv
import sqlite3
from pathlib import Path

import streamlit as st
import pandas as pd

DB_PATH = Path(__file__).with_name("kirana.db")

st.set_page_config(page_title="Kirana Billing App", page_icon="🛒", layout="wide")

def db():
    conn = sqlite3.connect(DB_PATH, check_same_thread=False)
    conn.row_factory = sqlite3.Row
    return conn

def init_db():
    with db() as conn:
        conn.execute("""CREATE TABLE IF NOT EXISTS products (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL UNIQUE,
            price REAL NOT NULL CHECK(price >= 0),
            stock INTEGER NOT NULL DEFAULT 0 CHECK(stock >= 0)
        )""")
        conn.execute("""CREATE TABLE IF NOT EXISTS sales (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            created_at TEXT NOT NULL,
            customer TEXT NOT NULL DEFAULT 'Walk-in customer',
            total REAL NOT NULL
        )""")
        conn.execute("""CREATE TABLE IF NOT EXISTS sale_items (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            sale_id INTEGER NOT NULL,
            product_id INTEGER,
            product_name TEXT NOT NULL,
            quantity INTEGER NOT NULL,
            unit_price REAL NOT NULL,
            line_total REAL NOT NULL,
            FOREIGN KEY(sale_id) REFERENCES sales(id)
        )""")
        count = conn.execute("SELECT COUNT(*) FROM products").fetchone()[0]
        if count == 0:
            conn.executemany(
                "INSERT INTO products(name, price, stock) VALUES (?, ?, ?)",
                [
                    ("Rice (1 kg)", 60.0, 100),
                    ("Milk (500 ml)", 30.0, 50),
                    ("Sugar (1 kg)", 45.0, 40),
                    ("Biscuits (pack)", 10.0, 120),
                    ("Cooking Oil (1 L)", 140.0, 25),
                    ("Tea Powder (250 g)", 85.0, 30),
                ],
            )

init_db()

st.markdown("""
<style>
    .block-container {padding-top: 1.6rem; padding-bottom: 2rem;}
    .hero {padding: 1.25rem 1.5rem; border-radius: 18px; background: linear-gradient(120deg,#12372a,#436850); color: white; margin-bottom: 1rem;}
    .hero h1 {color: white; margin: 0;}
    .hero p {margin: .35rem 0 0; opacity: .9;}
    div[data-testid="stMetric"] {border: 1px solid #e5e7eb; padding: 14px; border-radius: 12px;}
</style>
""", unsafe_allow_html=True)

st.markdown('<div class="hero"><h1>🛒 Kirana Billing App</h1><p>Simple billing and sales tracking for your neighbourhood shop.</p></div>', unsafe_allow_html=True)

with st.sidebar:
    st.title("Navigation")
    page = st.radio("Choose a page", ["New Bill", "Products & Stock", "Sales Dashboard", "About"])
    st.divider()
    st.caption("Demo app • Data is stored in a local SQLite database.")

def get_products():
    with db() as conn:
        return pd.read_sql_query("SELECT id, name, price, stock FROM products ORDER BY name", conn)

def money(value):
    return f"₹{float(value):,.2f}"

if page == "New Bill":
    st.subheader("Create a bill")
    products = get_products()
    if products.empty:
        st.info("Add products first from Products & Stock.")
    else:
        product_names = products["name"].tolist()
        product_map = {row["name"]: row for _, row in products.iterrows()}
        if "cart" not in st.session_state:
            st.session_state.cart = []
        left, right = st.columns([1.1, 1])
        with left:
            st.markdown("#### Add items")
            selected = st.selectbox("Product", product_names)
            row = product_map[selected]
            st.caption(f"Unit price: {money(row['price'])} • Available stock: {int(row['stock'])}")
            qty = st.number_input("Quantity", min_value=1, max_value=max(1, int(row["stock"]) or 1), value=1, step=1)
            if st.button("＋ Add to bill", use_container_width=True, type="primary"):
                existing = next((item for item in st.session_state.cart if item["product_id"] == int(row["id"])), None)
                already = existing["quantity"] if existing else 0
                if int(qty) + already > int(row["stock"]):
                    st.error("Not enough stock for that quantity.")
                elif int(row["stock"]) <= 0:
                    st.error("This product is out of stock.")
                elif existing:
                    existing["quantity"] += int(qty)
                    st.rerun()
                else:
                    st.session_state.cart.append({
                        "product_id": int(row["id"]), "name": row["name"],
                        "price": float(row["price"]), "quantity": int(qty)
                    })
                    st.rerun()
        with right:
            st.markdown("#### Customer")
            customer = st.text_input("Customer name (optional)", placeholder="Walk-in customer")
            st.markdown("#### Current bill")
            if st.session_state.cart:
                cart_df = pd.DataFrame([{
                    "Product": x["name"], "Qty": x["quantity"],
                    "Unit price": money(x["price"]), "Amount": money(x["price"] * x["quantity"])
                } for x in st.session_state.cart])
                st.dataframe(cart_df, use_container_width=True, hide_index=True)
                total = sum(x["price"] * x["quantity"] for x in st.session_state.cart)
                st.metric("Grand total", money(total))
                c1, c2 = st.columns(2)
                with c1:
                    if st.button("Clear bill", use_container_width=True):
                        st.session_state.cart = []
                        st.rerun()
                with c2:
                    if st.button("Complete sale", type="primary", use_container_width=True):
                        # Re-check stock at checkout to prevent overselling.
                        with db() as conn:
                            for item in st.session_state.cart:
                                stock = conn.execute("SELECT stock FROM products WHERE id=?", (item["product_id"],)).fetchone()
                                if stock is None or stock["stock"] < item["quantity"]:
                                    st.error(f"Insufficient stock for {item['name']}. Refresh and try again.")
                                    st.stop()
                            created = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                            cur = conn.execute(
                                "INSERT INTO sales(created_at, customer, total) VALUES (?, ?, ?)",
                                (created, customer.strip() or "Walk-in customer", total)
                            )
                            sale_id = cur.lastrowid
                            for item in st.session_state.cart:
                                line_total = item["price"] * item["quantity"]
                                conn.execute("""INSERT INTO sale_items
                                    (sale_id, product_id, product_name, quantity, unit_price, line_total)
                                    VALUES (?, ?, ?, ?, ?, ?)""",
                                    (sale_id, item["product_id"], item["name"], item["quantity"], item["price"], line_total))
                                conn.execute("UPDATE products SET stock = stock - ? WHERE id=?",
                                             (item["quantity"], item["product_id"]))
                        st.session_state.last_sale_id = sale_id
                        st.session_state.cart = []
                        st.success(f"Sale #{sale_id} saved successfully!")
                        st.rerun()
            else:
                st.info("Your bill is empty. Add a product to begin.")

    if st.session_state.get("last_sale_id"):
        sale_id = st.session_state.last_sale_id
        with db() as conn:
            sale = conn.execute("SELECT * FROM sales WHERE id=?", (sale_id,)).fetchone()
            items = conn.execute("SELECT product_name, quantity, unit_price, line_total FROM sale_items WHERE sale_id=?", (sale_id,)).fetchall()
        if sale:
            st.divider()
            st.markdown(f"### Latest receipt — Sale #{sale_id}")
            st.write(f"**KIRANA STORE**  ·  {sale['created_at']}")
            st.write(f"Customer: {sale['customer']}")
            receipt_df = pd.DataFrame([{
                "Item": i["product_name"], "Qty": i["quantity"],
                "Price": money(i["unit_price"]), "Amount": money(i["line_total"])
            } for i in items])
            st.dataframe(receipt_df, use_container_width=True, hide_index=True)
            st.markdown(f"## Total: {money(sale['total'])}")
            csv_bytes = receipt_df.to_csv(index=False).encode("utf-8")
            st.download_button("Download receipt (CSV)", data=csv_bytes, file_name=f"receipt_{sale_id}.csv", mime="text/csv")

elif page == "Products & Stock":
    st.subheader("Products & stock")
    with st.expander("＋ Add a new product", expanded=True):
        with st.form("add_product", clear_on_submit=True):
            name = st.text_input("Product name")
            c1, c2 = st.columns(2)
            price = c1.number_input("Price (₹)", min_value=0.0, value=10.0, step=1.0)
            stock = c2.number_input("Opening stock", min_value=0, value=10, step=1)
            submitted = st.form_submit_button("Add product", type="primary")
            if submitted:
                if not name.strip():
                    st.error("Please enter a product name.")
                else:
                    try:
                        with db() as conn:
                            conn.execute("INSERT INTO products(name, price, stock) VALUES (?, ?, ?)",
                                         (name.strip(), float(price), int(stock)))
                        st.success("Product added.")
                        st.rerun()
                    except sqlite3.IntegrityError:
                        st.error("A product with that name already exists.")
    products = get_products()
    st.dataframe(products.rename(columns={"id":"ID","name":"Product","price":"Price (₹)","stock":"Stock"}),
                 use_container_width=True, hide_index=True)
    st.markdown("#### Update price or stock")
    if not products.empty:
        chosen = st.selectbox("Select product to update", products["name"].tolist())
        selected_row = products[products["name"] == chosen].iloc[0]
        with st.form("update_product"):
            new_price = st.number_input("New price (₹)", min_value=0.0, value=float(selected_row["price"]), step=1.0)
            new_stock = st.number_input("New stock quantity", min_value=0, value=int(selected_row["stock"]), step=1)
            if st.form_submit_button("Save changes"):
                with db() as conn:
                    conn.execute("UPDATE products SET price=?, stock=? WHERE id=?",
                                 (float(new_price), int(new_stock), int(selected_row["id"])))
                st.success("Product updated.")
                st.rerun()

elif page == "Sales Dashboard":
    st.subheader("Sales dashboard")
    with db() as conn:
        sales = pd.read_sql_query("SELECT id, created_at, customer, total FROM sales ORDER BY id DESC", conn)
        items = pd.read_sql_query("SELECT product_name, quantity, line_total, sale_id FROM sale_items", conn)
    today = date.today().isoformat()
    today_sales = sales[sales["created_at"].str.startswith(today)] if not sales.empty else sales
    total_today = today_sales["total"].sum() if not today_sales.empty else 0
    col1, col2, col3 = st.columns(3)
    col1.metric("Sales today", len(today_sales))
    col2.metric("Revenue today", money(total_today))
    col3.metric("All-time revenue", money(sales["total"].sum() if not sales.empty else 0))
    if not sales.empty:
        st.markdown("#### Recent sales")
        show = sales.copy()
        show["total"] = show["total"].map(money)
        st.dataframe(show.rename(columns={"id":"Sale ID","created_at":"Date & time","customer":"Customer","total":"Total"}),
                     use_container_width=True, hide_index=True)
        st.download_button("Download sales report (CSV)", data=sales.to_csv(index=False).encode("utf-8"),
                           file_name="sales_report.csv", mime="text/csv")
        if not items.empty:
            st.markdown("#### Best-selling items")
            popular = items.groupby("product_name", as_index=False).agg(Quantity=("quantity","sum"), Revenue=("line_total","sum"))
            popular = popular.sort_values("Quantity", ascending=False).head(10)
            st.bar_chart(popular.set_index("product_name")["Quantity"])
            popular["Revenue"] = popular["Revenue"].map(money)
            st.dataframe(popular, use_container_width=True, hide_index=True)
    else:
        st.info("No sales recorded yet. Create your first bill from the New Bill page.")

else:
    st.subheader("About this project")
    st.write("""
    **Voice-Powered Kirana Billing App** is a lightweight billing and stock management tool for small Indian grocery shops.

    **Features**
    - Product catalogue with prices and stock quantities
    - Shopping cart with automatic totals
    - Checkout, saved sales, and itemized receipts
    - Daily revenue and sales dashboard
    - CSV downloads for receipts and sales reports
    - Local SQLite storage

    **Challenge demo tip:** Use Wispr Flow to dictate feature requests and follow-up changes into your coding assistant, then record the real app while creating a bill.
    """)
    st.warning("This is a demo project. Back up the local database before deleting or replacing the app files. For production use, add authentication, tax settings, receipt printing, backups, and proper deployment storage.")
''',
"requirements.txt": "streamlit>=1.37,<2.0\npandas>=2.0,<3.0\n",
"README.md": r'''# Voice-Powered Kirana Billing App

A simple Streamlit + SQLite billing application for small Indian grocery shops. It includes product/stock management, bill creation, saved sales, receipts, a sales dashboard, and CSV exports.

## Features

- Add products, prices, and opening stock
- Create a bill by adding products and quantities
- Automatic line totals and grand total in Indian rupees
- Checkout saves a sale and deducts stock
- Receipt preview and CSV download
- Daily sales metrics and all-time revenue
- Best-selling items chart
- Sales report CSV export
- Local SQLite database (`kirana.db`) created automatically

## Run locally

Requires Python 3.10 or newer.

```bash
python -m venv .venv
```

Activate the environment:

**Windows**
```powershell
.venv\Scripts\activate
```

**macOS/Linux**
```bash
source .venv/bin/activate
```

Install dependencies and start the app:

```bash
pip install -r requirements.txt
streamlit run app.py
```

The app opens in your browser, usually at `http://localhost:8501`.

## Deploy on Streamlit Community Cloud

1. Create a GitHub repository and upload `app.py`, `requirements.txt`, and `README.md`.
2. Open [Streamlit Community Cloud](https://share.streamlit.io/) and connect GitHub.
3. Choose the repository and set the main file path to `app.py`.
4. Deploy.

**Important:** The included SQLite database is local to the running environment. Some cloud deployments may reset local files when the app restarts or redeploys. For durable production data, connect a managed database.

## Suggested demo flow

1. Open **Products & Stock** and show sample products.
2. Go to **New Bill**, add rice, milk, and biscuits.
3. Show the automatically calculated total.
4. Complete the sale and show the receipt.
5. Open **Sales Dashboard** to show today's revenue and sales.
6. Explain how Wispr Flow was used to dictate requirements and follow-up changes during development.

## Wispr Flow challenge note

Only claim usage that actually happened. For the submission, record your real screen and include your actual Wispr usage card. Do not use a fabricated UI or fake usage metrics.

## Future improvements

- User login and role-based access
- GST/tax and discounts
- Printable PDF receipts
- Barcode scanning
- Customer credit/ledger
- Persistent managed database and automated backups
- Multilingual voice-based order entry
''',
".gitignore": "__pycache__/\n*.py[cod]\n.venv/\nvenv/\nk irana.db\nkirana.db\n.streamlit/secrets.toml\n",
"LICENSE": "MIT License\n\nCopyright (c) 2026\n\nPermission is hereby granted, free of charge, to any person obtaining a copy\nof this software and associated documentation files (the \"Software\"), to deal\nin the Software without restriction, including without limitation the rights\nto use, copy, modify, merge, publish, distribute, sublicense, and/or sell\ncopies of the Software, and to permit persons to whom the Software is\nfurnished to do so, subject to the following conditions:\n\nThe above copyright notice and this permission notice shall be included in all\ncopies or substantial portions of the Software.\n\nTHE SOFTWARE IS PROVIDED \"AS IS\", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR\nIMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,\nFITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.\n",
}
for name, content in files.items():
    (base / name).write_text(content, encoding="utf-8")

zip_path = pathlib.Path("/mnt/data/voice-powered-kirana-billing-app.zip")
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as zf:
    for p in base.rglob("*"):
        if p.is_file():
            zf.write(p, p.relative_to(base.parent))
print(f"Created {zip_path} ({zip_path.stat().st_size} bytes)")
