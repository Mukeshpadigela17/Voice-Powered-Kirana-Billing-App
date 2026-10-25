# 🛒 Voice-Powered Kirana Billing App

A simple and user-friendly billing and inventory management application designed for small Indian grocery shops (kirana stores). The app helps shopkeepers manage products, generate customer bills, track stock, and monitor daily sales efficiently.

## 🚀 Features

- Product Management: Add products, update prices, and manage stock quantities.
- Smart Billing: Add products to a cart and calculate totals automatically.
- Sales Management: Save completed sales and maintain transaction records.
- Digital Receipts: Generate itemized receipts and download them as CSV files.
- Inventory Tracking: Automatically update stock after each sale.
- Sales Dashboard: Monitor daily sales, revenue, and all-time earnings.
- Best-Selling Products: Visualize popular products based on sales quantity.
- CSV Reports: Download sales reports for record keeping.
- Sample Products: Explore the app immediately with preloaded grocery items.

## 🛠️ Tech Stack

- Programming Language: Python
- Frontend: Streamlit
- Database: SQLite
- Data Processing: Pandas
- Visualization: Streamlit Charts
- Development Tools: VS Code, Git, GitHub
- Voice-Driven Development: Wispr Flow with an AI coding assistant

## 📂 Project Structure

text
voice-powered-kirana-billing-app/

├── app.py

├── requirements.txt

├── README.md

├── LICENSE

└── .gitignore


The SQLite database (`kirana.db`) is created automatically when the application runs.

## ⚙️ Installation and Setup

### Prerequisites

- Python 3.10 or later
- Git (optional)
- A code editor such as VS Code

### Step 1: Clone the repository

bash
git clone https://github.com/YOUR_USERNAME/voice-powered-kirana-billing-app.git
cd voice-powered-kirana-billing-app


Replace `YOUR_USERNAME` with your GitHub username.

### Step 2: Create a virtual environment

bash
python -m venv .venv


Activate it on Windows:

bash
.venv\Scripts\activate


On macOS or Linux:

bash
source .venv/bin/activate


### Step 3: Install dependencies

bash
pip install -r requirements.txt


### Step 4: Run the application

bash
streamlit run app.py


Open the local URL displayed in your terminal, usually:

`http://localhost:8501`

## 📖 How to Use

1. Open Products & Stock to view or add grocery products.
2. Select New Bill to create a customer order.
3. Choose products and quantities to calculate the bill automatically.
4. Enter the customer's name if required.
5. Click Complete Sale to save the transaction and update inventory.
6. View the receipt and download it as a CSV file.
7. Open Sales Dashboard to review revenue, transactions, and popular products.

## 🎙️ Built with Wispr Flow

Wispr Flow enables voice-driven development by allowing developers to dictate requirements and follow-up instructions into their coding tools.

For this project, it can be used to describe features, request UI improvements, and refine the billing workflow through natural speech.

Example voice prompt:

> Build a simple kirana billing app with product management, a shopping cart, automatic total calculation, inventory updates, receipt generation, and a daily sales dashboard using Python and Streamlit.

## 🎯 Problem Statement

Many small grocery shop owners still depend on manual calculations and handwritten bills. This can make billing slower, increase calculation errors, and complicate stock tracking.

The Voice-Powered Kirana Billing App provides a simple digital solution to streamline daily billing, maintain inventory, and track sales.

## 🔮 Future Enhancements

- Voice-based order entry in Telugu, Hindi, and English
- Printable PDF receipts
- GST and discount calculations
- Barcode scanning
- Customer credit and payment tracking
- User authentication and access control
- Cloud database and automated backups
- Mobile-friendly interface

## ⚠️ Limitations

This project is a prototype intended for demonstration and learning. It uses a local SQLite database, so persistent storage should be configured before production deployment. Features such as GST invoicing, authentication, and PDF printing are not currently implemented.

## 🤝 Contributing

Contributions and suggestions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Commit your changes.
4. Open a pull request.

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

Built to make everyday kirana shop billing simpler, faster, and more efficient. 🛍️
