import streamlit as st
import pandas as pd
import sqlite3
import plotly.express as px
import time

# --- 1. إعدادات الصفحة والتصميم العام ---
st.set_page_config(page_title="نظام الدفاع وإدارة القوى البشرية", layout="wide", page_icon="🛡️")

# إضافة CSS مخصص ليظهر كأنه تطبيق حقيقي وليس صفحة ويب عادية
st.markdown("""
<style>
    [data-testid="stSidebar"] {background-color: #1E1E1E; color: white;}
    .stMetric {background-color: #f0f2f6; padding: 10px; border-radius: 5px; border-left: 5px solid #1f77b4;}
    h1, h2, h3 {font-family: 'Arial', sans-serif;}
</style>
""", unsafe_allow_html=True)

# --- 2. تهيئة قاعدة البيانات ---
conn = sqlite3.connect('military_pro.db', check_same_thread=False)
c = conn.cursor()
c.execute('CREATE TABLE IF NOT EXISTS soldiers (id TEXT PRIMARY KEY, name TEXT, rank TEXT, unit TEXT, status TEXT)')
c.execute('CREATE TABLE IF NOT EXISTS leaves (s_id TEXT, type TEXT, days INTEGER, status TEXT, date TEXT)')
conn.commit()

# --- 3. نظام تسجيل الدخول (Simulation) ---
if 'logged_in' not in st.session_state:
    st.session_state.logged_in = False

def login():
    st.markdown("<h1 style='text-align: center; color: #2C3E50;'>🛡️ بوابة الأمن والدفاع</h1>", unsafe_allow_html=True)
    st.markdown("<h3 style='text-align: center;'>نظام إدارة الموارد البشرية العسكرية</h3>", unsafe_allow_html=True)
    
    col1, col2, col3 = st.columns([1,2,1])
    with col2:
        with st.form("login_form"):
            username = st.text_input("اسم المستخدم (User ID)")
            password = st.text_input("كلمة المرور (Password)", type="password")
            submit = st.form_submit_button("تسجيل الدخول الآمن")
            
            if submit:
                # محاكاة الدخول (أي شيء سيقبله للتجربة)
                if username == "admin" and password == "1234":
                    st.session_state.logged_in = True
                    st.rerun()
                elif username != "" and password != "":
                     st.session_state.logged_in = True
                     st.rerun()
                else:
                    st.error("الرجاء إدخال بيانات الاعتماد")

if not st.session_state.logged_in:
    login()
    st.stop() # توقف هنا إذا لم يسجل الدخول

# --- 4. واجهة التطبيق الرئيسية (بعد الدخول) ---

# القائمة الجانبية
with st.sidebar:
    st.image("https://cdn-icons-png.flaticon.com/512/9203/9203764.png", width=100)
    st.title(f"القائد: {st.session_state.get('user', 'المدير العام')}")
    st.markdown("---")
    menu = st.radio("القائمة الرئيسية", 
        ["📊 لوحة القيادة (Dashboard)", "🪖 كتيبة الجنود", "📝 مركز الإجازات", "⚙️ الإعدادات"])
    
    st.markdown("---")
    if st.button("تسجيل الخروج"):
        st.session_state.logged_in = False
        st.rerun()

# --- الصفحة 1: لوحة القيادة ---
if menu == "📊 لوحة القيادة (Dashboard)":
    st.title("📊 مركز العمليات المشتركة")
    st.markdown("حالة القوة البشرية والجاهزية القتالية")
    
    # مؤشرات الأداء (KPIs)
    c.execute("SELECT count(*) FROM soldiers")
    total = c.fetchone()[0]
    c.execute("SELECT count(*) FROM leaves WHERE status='Approved'")
    on_leave = c.fetchone()[0]
    
    m1, m2, m3, m4 = st.columns(4)
    m1.metric("إجمالي القوة", f"{total}", "+2 هذا الأسبوع")
    m2.metric("جاهزية الميدان", f"{total - on_leave}", "95%")
    m3.metric("في إجازة رسمية", f"{on_leave}", "-1")
    m4.metric("تنبيهات أمنية", "0", "مستقر", delta_color="normal")
    
    # الرسم البياني (توزيع الرتب)
    st.divider()
    col_chart1, col_chart2 = st.columns(2)
    
    with col_chart1:
        st.subheader("توزيع الرتب العسكرية")
        df_ranks = pd.read_sql_query("SELECT rank, count(*) as count FROM soldiers GROUP BY rank", conn)
        if not df_ranks.empty:
            fig = px.pie(df_ranks, values='count', names='rank', hole=0.4, color_discrete_sequence=px.colors.sequential.RdBu)
            st.plotly_chart(fig, use_container_width=True)
        else:
            st.info("لا توجد بيانات كافية للرسم البياني")

    with col_chart2:
        st.subheader("حالة الإجازات الشهرية")
        # بيانات وهمية للعرض الجميل
        chart_data = pd.DataFrame({
            'الشهر': ['يناير', 'فبراير', 'مارس', 'أبريل'],
            'إجازات': [10, 15, 8, 12],
            'مرضية': [2, 3, 1, 4]
        })
        fig2 = px.bar(chart_data, x='الشهر', y=['إجازات', 'مرضية'], barmode='group')
        st.plotly_chart(fig2, use_container_width=True)

# --- الصفحة 2: كتيبة الجنود ---
elif menu == "🪖 كتيبة الجنود":
    st.title("🪖 إدارة ملفات الأفراد")
    
    tab1, tab2 = st.tabs(["سجل الجنود", "إضافة جندي جديد"])
    
    with tab1:
        st.markdown("#### قاعدة بيانات الأفراد")
        df = pd.read_sql_query("SELECT * FROM soldiers", conn)
        # تلوين الجدول ليصبح تفاعلياً
        st.dataframe(df, use_container_width=True, hide_index=True)
    
    with tab2:
        st.markdown("#### تسجيل فرد جديد")
        with st.form("add_soldier"):
            c1, c2 = st.columns(2)
            s_id = c1.text_input("الرقم العسكري")
            s_name = c2.text_input("الاسم الكامل")
            s_rank = c1.selectbox("الرتبة", ["جندي", "عريف", "رقيب", "ملازم", "نقيب", "رائد"])
            s_unit = c2.selectbox("الوحدة / الكتيبة", ["المشاة الأولى", "الدعم اللوجستي", "الإشارة", "العمليات الخاصة"])
            
            if st.form_submit_button("حفظ في السجلات"):
                c.execute("INSERT OR REPLACE INTO soldiers VALUES (?, ?, ?, ?, ?)", (s_id, s_name, s_rank, s_unit, 'في الخدمة'))
                conn.commit()
                st.success("تمت إضافة الجندي بنجاح")
                time.sleep(1)
                st.rerun()

# --- الصفحة 3: مركز الإجازات ---
elif menu == "📝 مركز الإجازات":
    st.title("📝 إدارة الموافقات والإجازات")
    
    c1, c2 = st.columns([1, 2])
    
    with c1:
        st.info("نظام تقديم الطلبات")
        with st.form("leave_request"):
            ls_id = st.text_input("الرقم العسكري لصاحب الطلب")
            l_type = st.selectbox("نوع التصريح", ["إجازة سنوية", "إجازة مرضية", "إذن خروج مؤقت"])
            l_days = st.number_input("المدة (بالأيام)", min_value=1, max_value=30)
            
            if st.form_submit_button("إرسال للموافقة"):
                # التحقق من وجود الجندي
                c.execute("SELECT name FROM soldiers WHERE id=?", (ls_id,))
                result = c.fetchone()
                if result:
                    current_date = time.strftime("%Y-%m-%d")
                    c.execute("INSERT INTO leaves VALUES (?, ?, ?, ?, ?)", (ls_id, l_type, l_days, 'Pending', current_date))
                    conn.commit()
                    st.success(f"تم رفع طلب للجندي: {result[0]}")
                else:
                    st.error("الرقم العسكري غير موجود في النظام!")

    with c2:
        st.subheader("الطلبات قيد الانتظار")
        # عرض الطلبات مع إمكانية اتخاذ قرار (محاكاة)
        leaves_df = pd.read_sql_query("SELECT * FROM leaves ORDER BY date DESC", conn)
        if not leaves_df.empty:
            for index, row in leaves_df.iterrows():
                if row['status'] == 'Pending':
                    with st.expander(f"طلب من الجندي {row['s_id']} - {row['type']}"):
                        st.write(f"المدة: {row['days']} يوم")
                        col_ok, col_no = st.columns(2)
                        if col_ok.button("موافقة ✅", key=f"ok_{index}"):
                            c.execute("UPDATE leaves SET status='Approved' WHERE s_id=? AND type=?", (row['s_id'], row['type']))
                            conn.commit()
                            st.rerun()
                        if col_no.button("رفض ❌", key=f"no_{index}"):
                            c.execute("UPDATE leaves SET status='Rejected' WHERE s_id=? AND type=?", (row['s_id'], row['type']))
                            conn.commit()
                            st.rerun()
        
        st.markdown("---")
        st.caption("سجل الموافقات التاريخي")
        st.dataframe(leaves_df)

# --- الصفحة 4: الإعدادات ---
elif menu == "⚙️ الإعدادات":
    st.header("إعدادات النظام")
    st.warning("هذه المنطقة مخصصة للمدير التقني فقط.")
    st.toggle("تفعيل الوضع الليلي (Dark Mode)")
    st.toggle("إيقاف استقبال طلبات الإجازة")
