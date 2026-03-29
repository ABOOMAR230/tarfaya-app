import streamlit as st
import pandas as pd
from datetime import datetime
from streamlit_folium import st_folium
import folium
import urllib.parse

# --- إعدادات هامة جداً ---
# استبدل الأصفار برقم موبايلك (يبدأ بـ 20) لاستقبال الطلبات عليه
MY_PHONE_NUMBER = "201026410452" 

st.set_page_config(page_title="تطبيق كسح الطرفاية", layout="centered")

# مخزن مؤقت للطلبات
if 'orders' not in st.session_state:
    st.session_state.orders = []

# قائمة الأسعار حسب المنطقة في البدرشين
pricing_map = {
    "داخل قرية الطرفاية": 150,
    "عزبة مدكور": 200,
    "قرية الشوبك": 250,
    "البدرشين (المركز)": 300,
    "المرازيق": 300
}

# التنقل الجانبي
page = st.sidebar.radio("القائمة:", ["طلب سيارة (للجمهور)", "لوحة التحكم (لصاحب المشروع)"])

if page == "طلب سيارة (للجمهور)":
    st.title("🚛 خدمة كسح الطرفاية")
    st.markdown("### اطلب الآن وسيتم توجيهك للواتساب فوراً")
    
    name = st.text_input("اسمك")
    phone = st.text_input("رقم تليفونك")
    
    st.write("📍 اضغط على موقع منزلك في الخريطة:")
    # إحداثيات تقريبية لمنطقة البدرشين/الطرفاية
    m = folium.Map(location=[29.8247, 31.2589], zoom_start=14)
    m.add_child(folium.LatLngPopup())
    map_data = st_folium(m, height=300, width=None)

    lat, lon = None, None
    if map_data['last_clicked']:
        lat = map_data['last_clicked']['lat']
        lon = map_data['last_clicked']['lng']
        st.success("تم تحديد الموقع بنجاح!")

    area = st.selectbox("اختر المنطقة:", list(pricing_map.keys()))
    
    if st.button("تأكيد الطلب ✅"):
        if name and phone and lat:
            g_maps = f"https://www.google.com/maps?q={lat},{lon}"
            msg = f"طلب كسح جديد:\n👤 العميل: {name}\n📞 هاتف: {phone}\n📍 المنطقة: {area}\n💰 السعر: {pricing_map[area]} ج.م\n🗺️ الموقع: {g_maps}"
            
            # رابط واتساب
            wa_url = f"https://wa.me/{MY_PHONE_NUMBER}?text={urllib.parse.quote(msg)}"
            
            # حفظ الطلب في الجلسة
            st.session_state.orders.append({"الوقت": datetime.now().strftime("%H:%M"), "العميل": name, "المنطقة": area})
            
            # توجيه العميل للواتساب
            st.markdown(f'<meta http-equiv="refresh" content="0;url={wa_url}">', unsafe_allow_html=True)
            st.info("جاري تحويلك للواتساب...")
        else:
            st.warning("من فضلك أكمل البيانات وحدد الموقع على الخريطة أولاً.")

else:
    st.title("📊 لوحة تحكم الإدارة")
    if st.session_state.orders:
        st.table(pd.DataFrame(st.session_state.orders))
        if st.button("مسح السجل"):
            st.session_state.orders = []
            st.rerun()
    else:
        st.write("لا توجد طلبات جديدة حالياً.")# tarfaya-app
