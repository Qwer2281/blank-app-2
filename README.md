import streamlit as st
import yfinance as yf
import plotly.graph_objects as go

st.title("📈 주식 거래 시뮬레이터")

# --- 주가 데이터 가져오기 ---
ticker = st.sidebar.text_input("종목코드 (예: AAPL, TSLA, MSFT)", "AAPL")
data = yf.download(ticker, period="6mo")

# --- 계좌 초기화 ---
if "cash" not in st.session_state:
    st.session_state.cash = 10000000  # 초기 자금 1천만원
    st.session_state.stocks = 0
    st.session_state.history = []

# --- 차트 표시 ---
fig = go.Figure()
fig.add_trace(go.Scatter(x=data.index, y=data["Close"], name="Close"))
st.plotly_chart(fig, use_container_width=True)

# --- 거래 기능 ---
price = data["Close"].iloc[-1]
st.sidebar.write(f"현재 가격: {price:.2f} USD")

shares = st.sidebar.number_input("매수/매도 수량", min_value=1, value=10)
if st.sidebar.button("매수"):
    cost = price * shares
    if st.session_state.cash >= cost:
        st.session_state.cash -= cost
        st.session_state.stocks += shares
        st.session_state.history.append((ticker, "매수", shares, price))
    else:
        st.sidebar.error("💸 잔액 부족!")

if st.sidebar.button("매도"):
    if st.session_state.stocks >= shares:
        st.session_state.cash += price * shares
        st.session_state.stocks -= shares
        st.session_state.history.append((ticker, "매도", shares, price))
    else:
        st.sidebar.error("📉 보유 주식 부족!")

# --- 현황 출력 ---
st.subheader("💰 계좌 현황")
st.write(f"현금: {st.session_state.cash:,.0f} 원")
st.write(f"보유 주식: {st.session_state.stocks} 주")

# --- 거래 내역 ---
st.subheader("📜 거래 내역")
st.table(st.session_state.history)
