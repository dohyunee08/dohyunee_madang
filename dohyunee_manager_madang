import streamlit as st 
import pymysql
import pandas as pd
import time

# DB 연결
dbConn = pymysql.connect(user='root', passwd='1234', host='localhost', db='madang', charset='utf8')
cursor = dbConn.cursor(pymysql.cursors.DictCursor)

def query(sql, params=None):
    cursor.execute(sql, params)
    return cursor.fetchall()

# 책 목록 불러오기
books = [None]
result = query("SELECT CONCAT(bookid, ',', bookname) AS info FROM Book;")
for res in result:
    books.append(res['info'])

tab1, tab2 = st.tabs(["고객조회", "거래 입력"])
name = tab1.text_input("고객명 입력")
custid = None

if len(name) > 0:
    # 1️⃣ 고객 조회
    result = query("SELECT * FROM Customer WHERE name=%s;", (name,))
    
    # 신규 고객이면 추가
    if not result:
        cursor.execute("INSERT INTO Customer (name) VALUES (%s);", (name,))
        dbConn.commit()
        st.success(f"신규 고객 '{name}'이 추가되었습니다.")
        result = query("SELECT * FROM Customer WHERE name=%s;", (name,))
    
    # 고객 정보 가져오기
    result_df = pd.DataFrame(result)
    tab1.write("고객 정보:")
    tab1.dataframe(result_df)
    
    custid = result_df['custid'][0]
    
    # 2️⃣ 거래 입력
    tab2.write("고객번호: " + str(custid))
    tab2.write("고객명: " + name)
    
    select_book = tab2.selectbox("구매 서적:", books)
    price = tab2.text_input("금액")
    
    if tab2.button('거래 입력'):
        if select_book and price:
            bookid = select_book.split(",")[0]
            dt = time.strftime('%Y-%m-%d', time.localtime())
            
            # 주문번호 계산
            max_order = query("SELECT IFNULL(MAX(orderid),0) AS max_id FROM Orders;")[0]['max_id']
            orderid = max_order + 1
            
            # 주문 등록
            cursor.execute(
                "INSERT INTO Orders (orderid, custid, bookid, saleprice, orderdate) VALUES (%s,%s,%s,%s,%s);",
                (orderid, custid, bookid, price, dt)
            )
            dbConn.commit()
            tab2.success(f"거래가 입력되었습니다. (주문번호: {orderid})")
