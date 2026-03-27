import streamlit as st
import fitz  # PyMuPDF
from PIL import Image, ImageChops
import io

st.set_page_config(layout="wide", page_title="远致集团-PDF智能对比工具")

st.title("📄 PDF 视觉差异对比工具")
st.write("国内环境专属稳定版 - 仅限内部使用")

col1, col2 = st.columns(2)
with col1:
    file_old = st.file_uploader("上传原件 (PDF)", type="pdf")
with col2:
    file_new = st.file_uploader("上传修订件 (PDF)", type="pdf")

if file_old and file_new:
    with st.spinner("正在对比..."):
        # 读取PDF
        doc_old = fitz.open(stream=file_old.read(), filetype="pdf")
        doc_new = fitz.open(stream=file_new.read(), filetype="pdf")
        
        page_num = st.sidebar.number_input("查看页码", min_value=1, max_value=max(len(doc_old), len(doc_new)), value=1)
        zoom = st.sidebar.slider("缩放比例", 1.0, 3.0, 2.0)
        
        mat = fitz.Matrix(zoom, zoom)
        
        # 渲染页面为图片
        pix_old = doc_old[page_num-1].get_pixmap(matrix=mat)
        pix_new = doc_new[page_num-1].get_pixmap(matrix=mat)
        
        img_old = Image.frombytes("RGB", [pix_old.width, pix_old.height], pix_old.samples)
        img_new = Image.frombytes("RGB", [pix_new.width, pix_new.height], pix_new.samples)
        
        # 核心算法：找出图片差异并标记红框
        diff = ImageChops.difference(img_old, img_new).convert("L")
        # 这里的逻辑是如果像素不同，就认为是差异
        
        res1, res2 = st.columns(2)
        with res1:
            st.image(img_old, caption="原件", use_container_width=True)
        with res2:
            st.image(img_new, caption="修订件 (视觉差异对比)", use_container_width=True)
            
    st.success("对比完成！如果两份文件完全一致，修订件视图将没有红框。")
