import streamlit as st
import requests

# 세션 상태 초기화
if "logged_in" not in st.session_state:
    st.session_state.logged_in = False
if "user_db" not in st.session_state:
    st.session_state.user_db = {
        "user": {
            "password": "1234",
            "nickname": "관리자",
            "town": "광주광역시 광산구 장덕로 9번길 73-7번지",
            "lat": None,
            "lon": None,
            "phone": "010-3267-7757",
            "image": None
        }
    }

st.set_page_config(page_title="우리동네 장애인 복지용구 플랫폼", layout="wide")

# ---------------------------- 로그인/회원가입 화면 ----------------------------
if not st.session_state.logged_in:
    st.markdown("# 🧡 우리동네 장애인 복지용구 중고거래 플랫폼 🧡")
    st.markdown("복지용구는 동네에서 가장 따뜻하게 나눌 수 있는 물건입니다. 지금 가입하고 시작해보세요!")

    tab1, tab2 = st.tabs(["🔐 로그인", "🆕 회원가입"])

    # 로그인 탭
    with tab1:
        login_username = st.text_input("아이디", key="login_id")
        login_password = st.text_input("비밀번호", type="password", key="login_pw")
        if st.button("로그인"):
            user_db = st.session_state.user_db
            if login_username in user_db and user_db[login_username]["password"] == login_password:
                st.success("로그인 성공!")
                st.session_state.logged_in = True
                st.session_state.current_user = login_username
                st.rerun()
            else:
                st.error("아이디 또는 비밀번호가 잘못되었습니다.")

    # 회원가입 탭
    with tab2:
        new_username = st.text_input("새 아이디", key="signup_id")
        new_password = st.text_input("새 비밀번호", type="password", key="signup_pw")
        confirm_password = st.text_input("비밀번호 확인", type="password", key="signup_confirm")
        nickname = st.text_input("닉네임")
        address = st.text_input("도로명 주소 입력 (예: 서울특별시 강남구 압구정동)")

        lat, lon, town_name = None, None, None
        if address:
            headers = {"Authorization": "0ed05dc29d17f60c718729a5b49f20cb"}
            kakao_url = f"https://dapi.kakao.com/v2/local/search/address.json?query={address}"
            res = requests.get(kakao_url, headers=headers)
            if res.status_code == 200 and res.json()["documents"]:
                lon = res.json()["documents"][0]["x"]
                lat = res.json()["documents"][0]["y"]
                st.success(f"📍 주소 좌표 확인 완료: 위도={lat}, 경도={lon}")

                reverse_url = f"https://dapi.kakao.com/v2/local/geo/coord2regioncode.json?x={lon}&y={lat}"
                res2 = requests.get(reverse_url, headers=headers)
                if res2.status_code == 200 and res2.json()["documents"]:
                    town_name = res2.json()["documents"][0]["region_3depth_name"]
                    st.markdown(f"📌 자동 설정된 동네: **{town_name}**")
            else:
                st.warning("주소 변환에 실패했습니다. 정확한 도로명 주소를 입력해주세요.")

        welfare_card = st.file_uploader("복지카드 또는 등록증 이미지 업로드", type=["jpg", "jpeg", "png", "pdf"])
        phone = st.text_input("휴대폰 번호 (예: 010-1234-5678)")
        if st.button("인증번호 전송"):
            st.info("📩 인증번호가 전송되었습니다. (※ 실제 구현 시 SMS API 필요)")
            # 여기에 인증번호 입력 및 확인 로직을 추가할 수 있음

        if st.button("회원가입"):
            if new_username in st.session_state.user_db:
                st.warning("이미 존재하는 아이디입니다.")
            elif new_password != confirm_password:
                st.warning("비밀번호가 일치하지 않습니다.")
            elif new_username == "" or new_password == "" or nickname == "" or address == "" or phone == "" or not lat or not lon:
                st.warning("모든 항목을 입력해주세요.")
            else:
                st.session_state.user_db[new_username] = {
                    "password": new_password,
                    "nickname": nickname,
                    "town": town_name or address,
                    "lat": lat,
                    "lon": lon,
                    "phone": phone,
                    "image": welfare_card
                }
                st.success("회원가입이 완료되었습니다! 이제 로그인해주세요.")

# ---------------------------- 로그인 후 본문 ----------------------------
else:
    user_info = st.session_state.user_db.get(st.session_state.current_user, {})
    st.sidebar.title("📋 메뉴")
    menu = st.sidebar.radio("", ["홈", "동네생활", "동네지도", "채팅", "마이페이지"])

    st.success(f"🎉 {user_info.get('nickname', '사용자')}님 환영합니다!")
    st.markdown(f"📍 현재 동네: **{user_info.get('town', '미입력')}**")

    if user_info.get("image"):
        st.image(user_info["image"], width=120, caption="프로필 사진")

    if menu == "홈":
        st.header("")
        st.markdown("여기에는 플랫폼의 요약 정보 또는 안내 문구를 배치할 예정")

    elif menu == "동네생활":
        st.header("")
        st.info("이웃과 소통하고 정보를 나눌 수 있는 공간입니다. (예정)")

    elif menu == "동네지도":
        st.header("")
        st.info("지도를 통해 거래 가능 복지용구를 확인할 수 있습니다. (예정)")

    elif menu == "채팅":
        st.header("")
        st.info("실시간 채팅 기능은 추후 제공될 예정입니다.")

    elif menu == "마이페이지":
        st.header("")
        st.info("내가 등록한 정보와 인증 상태를 확인할 수 있습니다.")

    st.sidebar.markdown("---")
    if st.sidebar.button("🔒 로그아웃"):
        st.session_state.logged_in = False
        st.rerun()
