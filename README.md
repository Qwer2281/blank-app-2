import streamlit as st
import yfinance as yf
import plotly.graph_objects as go
import pandas as pd
import requests

st.title("주식 거래 시뮬레이터")

# --- 환율 가져오기 (USD → KRW) ---
def get_usd_to_krw():
    try:
        url = "https://api.exchangerate.host/latest?base=USD&symbols=KRW"
        data = requests.get(url).json()
        return data["rates"]["KRW"]
    except:
        return 1300  # 기본값

usd_to_krw = get_usd_to_krw()

# --- 티커 선택 ---
ticker = st.sidebar.text_input("티커 입력", "AAPL")
data = yf.download(ticker, period="6mo")

# --- 티커별 세션 상태 초기화 ---
if "accounts" not in st.session_state:
    st.session_state.accounts = {}

if ticker not in st.session_state.accounts:
    st.session_state.accounts[ticker] = {
        "cash": 10000000,  # 초기 자금 1천만원
        "stocks": 0,
        "history": []
    }

account = st.session_state.accounts[ticker]

# --- 차트 표시 ---
fig = go.Figure()
fig.add_trace(go.Scatter(x=data.index, y=data["Close"], name="Close"))
st.plotly_chart(fig, use_container_width=True)

# --- 거래 기능 ---
price_usd = data["Close"].iloc[-1]
price_krw = price_usd * usd_to_krw
st.sidebar.write(f"현재 가격: ${price_usd:.2f} / 약 {price_krw:,.0f} 원")

shares = st.sidebar.number_input("매수/매도 수량", min_value=1, value=10)

if st.sidebar.button("매수"):
    cost = price_krw * shares
    if account["cash"] >= cost:
        account["cash"] -= cost
        account["stocks"] += shares
        account["history"].append([ticker, "매수", shares, price_krw])
        st.sidebar.success(f"{shares}주 매수 완료!")
    else:
        st.sidebar.error("잔액 부족")

if st.sidebar.button("매도"):
    if account["stocks"] >= shares:
        account["cash"] += price_krw * shares
        account["stocks"] -= shares
        account["history"].append([ticker, "매도", shares, price_krw])
        st.sidebar.success(f"{shares}주 매도 완료!")
    else:
        st.sidebar.error("보유 주식 부족")

# --- 계좌 현황 ---
st.subheader("💰 계좌 현황")
st.write(f"현금: {account['cash']:,.0f} 원")
st.write(f"보유 주식: {account['stocks']} 주")

# --- 거래 내역 ---
st.subheader("📜 거래 내역")
if account["history"]:
    st.table(pd.DataFrame(account["history"], columns=["티커", "거래", "수량", "가격(원)"]))
else:
    st.write("거래 내역이 없습니다.")
