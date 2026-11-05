import streamlit as st
import pandas as pd
import numpy as np

# 1. 페이지 설정 및 제목 (텍스트)
st.set_page_config(page_title="나만의 운동 기록 대시보드", page_icon="🏋️‍♂️", layout="wide")

st.title("🏋️‍♂️ 나만의 운동 기록 대시보드")
st.markdown("매일의 운동 기록을 확인하고, 운동 시간 통계를 분석해보세요!")
st.divider()

# ---------------------------------------------------------
# [나중에 수정할 곳] 운동 목록 설정
# ---------------------------------------------------------
exercise_list = ["스탠딩 크로스 크런치", "푸쉬업", "레그레이즈", "러닝", "스쿼트", "카프레이즈"]

# 2. 샘플 데이터 생성 (데이터 - 운동 종류, 운동 시간만 포함)
np.random.seed(42)
dates = pd.date_range(end=pd.Timestamp.now(), periods=14).strftime("%Y-%m-%d")

sample_data = []
for date in dates:
    ex = np.random.choice(exercise_list)
    duration = int(np.random.randint(15, 90))  # 운동 시간 (분)
    sample_data.append({"날짜": date, "운동종류": ex, "운동시간(분)": duration})

df = pd.DataFrame(sample_data)

# 3. 사이드바 위젯 구성 (위젯)
st.sidebar.header("🔍 데이터 필터링")

# 위젯 1: 운동 종류 선택 (selectbox)
selected_exercise = st.sidebar.selectbox("조회할 운동을 선택하세요:", ["전체"] + exercise_list)

# 위젯 2: 최소 운동 시간 선택 (slider)
min_duration = st.sidebar.slider("최소 운동 시간 (분):", min_value=0, max_value=120, value=0, step=5)

# 4. 위젯-화면 연동 (필터링 로직)
filtered_df = df.copy()

if selected_exercise != "전체":
    filtered_df = filtered_df[filtered_df["운동종류"] == selected_exercise]

filtered_df = filtered_df[filtered_df["운동시간(분)"] >= min_duration]

# 5. 메인 화면 구성 및 지표 표시 (데이터 - st.metric)
st.header("📊 운동 요약 정보")

col1, col2 = st.columns(2)

total_time = filtered_df["운동시간(분)"].sum()
total_count = len(filtered_df)

col1.metric(label="총 운동 횟수", value=f"{total_count} 회")
col2.metric(label="총 운동 시간", value=f"{total_time} 분")

st.divider()

# 6. 차트 구성 (차트 - st.bar_chart)
st.subheader("📅 일별 운동 시간 (분)")
if not filtered_df.empty:
    chart_data_time = filtered_df.set_index("날짜")[["운동시간(분)"]]
    st.bar_chart(chart_data_time)
else:
    st.info("조건에 맞는 데이터가 없습니다.")

st.divider()

# 7. 상세 데이터 테이블 표시 (데이터 - st.dataframe)
st.subheader("📋 세부 운동 기록 목록")
st.dataframe(filtered_df, use_container_width=True)
