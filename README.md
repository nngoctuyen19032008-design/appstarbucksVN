# appstarbucksVN
import streamlit as st
from datetime import datetime
import pandas as pd
import io
import html


# =========================================================
# CẤU HÌNH
# =========================================================

st.set_page_config(
    page_title="Tea Milk POS",
    page_icon="🧋",
    layout="wide"
)


# =========================================================
# DỮ LIỆU MENU
# =========================================================

MENU = {
    "Trà sữa truyền thống": 30000,
    "Trà sữa socola": 32000,
    "Trà sữa matcha": 35000,
    "Trà sữa khoai môn": 35000,
    "Trà sữa dâu": 33000,
    "Trà sữa thái xanh": 30000,
    "Trà sữa thái đỏ": 30000,
    "Trà đào": 28000,
    "Trà vải": 28000,
    "Trà chanh": 22000,
    "Trà tắc": 22000,
    "Matcha latte": 38000,
    "Cà phê sữa": 30000,
}

SIZE_PRICE = {
    "M": 0,
    "L": 5000,
    "XL": 10000,
}

TOPPING_PRICE = {
    "Không topping": 0,
    "Trân châu đen": 5000,
    "Trân châu trắng": 5000,
    "Thạch dừa": 5000,
    "Thạch trái cây": 5000,
    "Pudding trứng": 7000,
    "Kem cheese": 8000,
    "Trân châu hoàng kim": 7000,
}

SUGAR_LEVELS = [
    "100%",
    "70%",
    "50%",
    "30%",
    "0%"
]

ICE_LEVELS = [
    "100%",
    "70%",
    "50%",
    "30%",
    "0%"
]


# =========================================================
# SESSION STATE
# =========================================================

if "cart" not in st.session_state:
    st.session_state.cart = []

if "invoice" not in st.session_state:
    st.session_state.invoice = None

if "invoice_number" not in st.session_state:
    st.session_state.invoice_number = 1


# =========================================================
# HÀM TIỆN ÍCH
# =========================================================

def money(value):
    """Định dạng tiền Việt Nam."""
    return f"{value:,.0f} đ".replace(",", ".")


def calculate_item_price(drink, size, topping):
    """Tính giá một món."""
    return (
        MENU[drink]
        + SIZE_PRICE[size]
        + TOPPING_PRICE[topping]
    )


def create_invoice_text(invoice):
    """Tạo nội dung hóa đơn dạng text để tải xuống."""

    lines = []

    lines.append("=" * 50)
    lines.append("             HÓA ĐƠN TRÀ SỮA")
    lines.append("=" * 50)

    lines.append(f"Mã hóa đơn : {invoice['invoice_number']}")
    lines.append(f"Thời gian  : {invoice['datetime']}")
    lines.append(f"Khách hàng : {invoice['customer']}")
    lines.append("-" * 50)

    for i, item in enumerate(invoice["items"], 1):
        lines.append(
            f"{i}. {item['drink']} - Size {item['size']}"
        )

        if item["topping"] != "Không topping":
            lines.append(
                f"   Topping: {item['topping']}"
            )

        lines.append(
            f"   Đường: {item['sugar']} | Đá: {item['ice']}"
        )

        lines.append(
            f"   SL: {item['quantity']} x "
            f"{money(item['unit_price'])} = "
            f"{money(item['total'])}"
        )

    lines.append("-" * 50)
    lines.append(f"TỔNG TIỀN: {money(invoice['total'])}")
    lines.append(f"PHƯƠNG THỨC: {invoice['payment_method']}")
    lines.append("=" * 50)
    lines.append("       Cảm ơn quý khách! ❤️")
    lines.append("=" * 50)

    return "\n".join(lines)


def create_invoice_html(invoice):
    """Tạo hóa đơn HTML đẹp để tải xuống/in."""

    rows = ""

    for i, item in enumerate(invoice["items"], 1):

        topping = (
            ""
            if item["topping"] == "Không topping"
            else f"<br><small>Topping: {html.escape(item['topping'])}</small>"
        )

        rows += f"""
        <tr>
            <td>{i}</td>
            <td>
                <b>{html.escape(item['drink'])}</b>
                {topping}
                <br>
                <small>
                    Size {html.escape(item['size'])}
                    | Đường {html.escape(item['sugar'])}
                    | Đá {html.escape(item['ice'])}
                </small>
            </td>
            <td style="text-align:center">
                {item['quantity']}
            </td>
            <td style="text-align:right">
                {money(item['unit_price'])}
            </td>
            <td style="text-align:right">
                {money(item['total'])}
            </td>
        </tr>
        """

    html_content = f"""
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>Hóa đơn {invoice['invoice_number']}</title>

        <style>
            body {{
                font-family: Arial, sans-serif;
                max-width: 800px;
                margin: 30px auto;
                padding: 20px;
                color: #222;
            }}

            .header {{
                text-align: center;
                margin-bottom: 25px;
            }}

            .header h1 {{
                margin-bottom: 5px;
            }}

            .info {{
                margin-bottom: 20px;
            }}

            table {{
                width: 100%;
                border-collapse: collapse;
            }}

            th, td {{
                padding: 10px;
                border-bottom: 1px solid #ddd;
            }}

            th {{
                background: #f5f5f5;
            }}

            .total {{
                text-align: right;
                font-size: 22px;
                font-weight: bold;
                margin-top: 20px;
            }}

            .footer {{
                text-align: center;
                margin-top: 40px;
            }}

            @media print {{
                body {{
                    margin: 0;
                }}
            }}
        </style>
    </head>

    <body>

        <div class="header">
            <h1>🧋 TRÀ SỮA</h1>
            <div>HÓA ĐƠN THANH TOÁN</div>
        </div>

        <div class="info">
            <b>Mã hóa đơn:</b> {invoice['invoice_number']}<br>
            <b>Thời gian:</b> {invoice['datetime']}<br>
            <b>Khách hàng:</b> {html.escape(invoice['customer'])}
        </div>

        <table>
            <thead>
                <tr>
                    <th>#</th>
                    <th>Món</th>
                    <th>SL</th>
                    <th>Đơn giá</th>
                    <th>Thành tiền</th>
                </tr>
            </thead>

            <tbody>
                {rows}
            </tbody>
        </table>

        <div class="total">
            Tổng tiền: {money(invoice['total'])}
        </div>

        <p>
            <b>Phương thức thanh toán:</b>
            {html.escape(invoice['payment_method'])}
        </p>

        <div class="footer">
            Cảm ơn quý khách đã ủng hộ! ❤️<br>
            Hẹn gặp lại quý khách!
        </div>

    </body>
    </html>
    """

    return html_content


# =========================================================
# CSS
# =========================================================

st.markdown(
    """
    <style>

    .main-title {
        font-size: 42px;
        font-weight: 800;
        text-align: center;
        margin-bottom: 5px;
    }

    .sub-title {
        text-align: center;
        color: #777;
        margin-bottom: 30px;
    }

    .price-box {
        padding: 15px;
        border-radius: 12px;
        background: #f7f7f7;
        margin-top: 10px;
        font-size: 18px;
    }

    .total-box {
        padding: 20px;
        border-radius: 15px;
        background: #fff3f6;
        text-align: center;
        font-size: 28px;
        font-weight: bold;
        margin-top: 20px;
    }

    </style>
    """,
    unsafe_allow_html=True
)


# =========================================================
# HEADER
# =========================================================

st.markdown(
    '<div class="main-title">🧋 TEA MILK POS</div>',
    unsafe_allow_html=True
)

st.markdown(
    '<div class="sub-title">Hệ thống tính tiền & quản lý hóa đơn trà sữa</div>',
    unsafe_allow_html=True
)


# =========================================================
# THÔNG TIN KHÁCH HÀNG
# =========================================================

st.subheader("👤 Thông tin khách hàng")

customer_name = st.text_input(
    "Tên khách hàng",
    placeholder="Nhập tên khách hàng..."
)


# =========================================================
# CHỌN MÓN
# =========================================================

st.divider()

st.subheader("🥤 Thêm món")

col1, col2 = st.columns(2)

with col1:

    drink = st.selectbox(
        "Loại trà sữa / nước",
        list(MENU.keys())
    )

    size = st.selectbox(
        "Size",
        list(SIZE_PRICE.keys())
    )

    topping = st.selectbox(
        "Topping",
        list(TOPPING_PRICE.keys())
    )


with col2:

    sugar = st.select_slider(
        "Mức độ đường",
        options=SUGAR_LEVELS,
        value="70%"
    )

    ice = st.select_slider(
        "Mức độ đá",
        options=ICE_LEVELS,
        value="70%"
    )

    quantity = st.number_input(
        "Số lượng",
        min_value=1,
        max_value=50,
        value=1,
        step=1
    )


# =========================================================
# GIÁ MÓN
# =========================================================

unit_price = calculate_item_price(
    drink,
    size,
    topping
)

total_item_price = unit_price * quantity

st.markdown(
    f"""
    <div class="price-box">
        Giá mỗi ly: <b>{money(unit_price)}</b>
        &nbsp;&nbsp; | &nbsp;&nbsp;
        Thành tiền: <b>{money(total_item_price)}</b>
    </div>
    """,
    unsafe_allow_html=True
)


# =========================================================
# THÊM MÓN VÀO GIỎ
# =========================================================

if st.button(
    "➕ Thêm món vào hóa đơn",
    use_container_width=True
):

    item = {
        "drink": drink,
        "size": size,
        "topping": topping,
        "sugar": sugar,
        "ice": ice,
        "quantity": quantity,
        "unit_price": unit_price,
        "total": total_item_price
    }

    st.session_state.cart.append(item)

    st.success(
        f"Đã thêm {quantity} x {drink} vào hóa đơn!"
    )


# =========================================================
# HIỂN THỊ GIỎ HÀNG
# =========================================================

st.divider()

st.subheader("🛒 Hóa đơn hiện tại")


if len(st.session_state.cart) == 0:

    st.info(
        "Chưa có món nào. Hãy thêm món ở phía trên."
    )

else:

    cart_data = []

    for i, item in enumerate(
        st.session_state.cart
    ):

        cart_data.append(
            {
                "#": i + 1,
                "Món": item["drink"],
                "Size": item["size"],
                "Topping": item["topping"],
                "Đường": item["sugar"],
                "Đá": item["ice"],
                "SL": item["quantity"],
                "Đơn giá": money(item["unit_price"]),
                "Thành tiền": money(item["total"])
            }
        )

    df = pd.DataFrame(cart_data)

    st.dataframe(
        df,
        use_container_width=True,
        hide_index=True
    )

    cart_total = sum(
        item["total"]
        for item in st.session_state.cart
    )

    st.markdown(
        f"""
        <div class="total-box">
            💰 TỔNG TIỀN: {money(cart_total)}
        </div>
        """,
        unsafe_allow_html=True
    )


    # =====================================================
    # XÓA MÓN
    # =====================================================

    st.subheader("🗑️ Quản lý món")

    delete_options = [
        f"{i + 1}. {item['drink']} - "
        f"Size {item['size']} - "
        f"{item['quantity']} ly"
        for i, item in enumerate(
            st.session_state.cart
        )
    ]

    selected_delete = st.selectbox(
        "Chọn món muốn xóa",
        delete_options
    )

    delete_index = delete_options.index(
        selected_delete
    )

    if st.button(
        "🗑️ Xóa món đã chọn",
        use_container_width=True
    ):

        st.session_state.cart.pop(
            delete_index
        )

        st.rerun()


    # =====================================================
    # XÓA TOÀN BỘ
    # =====================================================

    if st.button(
        "🧹 Xóa toàn bộ hóa đơn",
        use_container_width=True
    ):

        st.session_state.cart = []

        st.rerun()


    # =====================================================
    # THANH TOÁN
    # =====================================================

    st.divider()

    st.subheader("💳 Thanh toán")

    payment_method = st.radio(
        "Phương thức thanh toán",
        [
            "💵 Tiền mặt",
            "🏦 Chuyển khoản",
            "💳 Thẻ ngân hàng"
        ],
        horizontal=True
    )

    if st.button(
        "✅ THANH TOÁN",
        type="primary",
        use_container_width=True
    ):

        if not customer_name.strip():
            st.error(
                "Vui lòng nhập tên khách hàng!"
            )

        else:

            final_total = sum(
                item["total"]
                for item in st.session_state.cart
            )

            invoice = {
                "invoice_number":
                    f"HD{datetime.now().strftime('%Y%m%d')}-"
                    f"{st.session_state.invoice_number:04d}",

                "datetime":
                    datetime.now().strftime(
                        "%d/%m/%Y %H:%M:%S"
                    ),

                "customer":
                    customer_name.strip(),

                "items":
                    st.session_state.cart.copy(),

                "total":
                    final_total,

                "payment_method":
                    payment_method
            }

            st.session_state.invoice = invoice

            st.session_state.invoice_number += 1

            st.session_state.cart = []

            st.success(
                "Thanh toán thành công! 🎉"
            )


# =========================================================
# HÓA ĐƠN SAU KHI THANH TOÁN
# =========================================================

if st.session_state.invoice:

    st.divider()

    st.header("🧾 HÓA ĐƠN ĐÃ THANH TOÁN")

    invoice = st.session_state.invoice

    col1, col2, col3 = st.columns(3)

    with col1:
        st.write(
            f"**Mã hóa đơn:** {invoice['invoice_number']}"
        )

    with col2:
        st.write(
            f"**Khách hàng:** {invoice['customer']}"
        )

    with col3:
        st.write(
            f"**Thời gian:** {invoice['datetime']}"
        )

    st.write("### Chi tiết")

    invoice_data = []

    for i, item in enumerate(
        invoice["items"], 1
    ):

        invoice_data.append(
            {
                "#": i,
                "Món": item["drink"],
                "Size": item["size"],
                "Topping": item["topping"],
                "Đường": item["sugar"],
                "Đá": item["ice"],
                "SL": item["quantity"],
                "Đơn giá": money(item["unit_price"]),
                "Thành tiền": money(item["total"])
            }
        )

    invoice_df = pd.DataFrame(
        invoice_data
    )

    st.dataframe(
        invoice_df,
        use_container_width=True,
        hide_index=True
    )

    st.markdown(
        f"""
        <div class="total-box">
            💰 TỔNG THANH TOÁN:
            {money(invoice['total'])}
        </div>
        """,
        unsafe_allow_html=True
    )

    st.write(
        f"**Phương thức:** {invoice['payment_method']}"
    )


    # =====================================================
    # XUẤT HÓA ĐƠN TXT
    # =====================================================

    invoice_text = create_invoice_text(
        invoice
    )

    st.download_button(
        label="📄 Xuất hóa đơn TXT",
        data=invoice_text,
        file_name=(
            f"{invoice['invoice_number']}.txt"
        ),
        mime="text/plain",
        use_container_width=True
    )


    # =====================================================
    # XUẤT HÓA ĐƠN HTML
    # =====================================================

    invoice_html = create_invoice_html(
        invoice
    )

    st.download_button(
        label="🧾 Xuất hóa đơn HTML / In",
        data=invoice_html,
        file_name=(
            f"{invoice['invoice_number']}.html"
        ),
        mime="text/html",
        use_container_width=True
    )


    # =====================================================
    # HÓA ĐƠN DẠNG CSV
    # =====================================================

    csv_buffer = io.StringIO()

    invoice_df.to_csv(
        csv_buffer,
        index=False,
        encoding="utf-8-sig"
    )

    st.download_button(
        label="📊 Xuất danh sách món CSV",
        data=csv_buffer.getvalue(),
        file_name=(
            f"{invoice['invoice_number']}.csv"
        ),
        mime="text/csv",
        use_container_width=True
    )


    # =====================================================
    # TẠO HÓA ĐƠN MỚI
    # =====================================================

    if st.button(
        "🆕 Tạo hóa đơn mới",
        use_container_width=True
    ):

        st.session_state.invoice = None

        st.rerun()
