import streamlit as st
import openai
import os
import requests
from PIL import Image
from io import BytesIO

# Đặt API key của bạn ở đây hoặc dùng biến môi trường
openai.api_key = os.getenv("OPENAI_API_KEY") or "YOUR_API_KEY_HERE"

st.title("🎨 Tạo Ảnh Hàng Loạt với Prompt")
st.markdown("Nhập danh sách prompt bên dưới, mỗi dòng là một mô tả ảnh.")

prompt_input = st.text_area("📋 Danh sách Prompt", height=200)

col1, col2, col3 = st.columns(3)

with col1:
    size_option = st.selectbox("🖼️ Kích thước ảnh", ["1024x1024", "1024x1792", "1792x1024"])

with col2:
    style_option = st.selectbox("🎨 Phong cách ảnh", ["Mặc định", "Phong cách minh họa", "Phong cách tranh sơn dầu", "Phong cách hoạt hình", "Ảnh hiện thực"])

with col3:
    format_option = st.selectbox("📁 Định dạng ảnh", ["PNG", "JPEG"])

button = st.button("🚀 Tạo Ảnh")

# Hàm xử lý style dạng thêm vào prompt
style_map = {
    "Mặc định": "",
    "Phong cách minh họa": ", illustration style",
    "Phong cách tranh sơn dầu": ", oil painting style",
    "Phong cách hoạt hình": ", cartoon style",
    "Ảnh hiện thực": ", realistic photo style"
}

if button and prompt_input:
    prompts = [p.strip() for p in prompt_input.split("\n") if p.strip()]

    for i, prompt in enumerate(prompts):
        full_prompt = prompt + style_map.get(style_option, "")
        with st.spinner(f"Đang tạo ảnh {i+1}/{len(prompts)}..."):
            try:
                response = openai.images.generate(
                    model="dall-e-3",
                    prompt=full_prompt,
                    n=1,
                    size=size_option
                )
                image_url = response.data[0].url

                # Tải ảnh về từ URL
                img_response = requests.get(image_url)
                image = Image.open(BytesIO(img_response.content))

                # Hiển thị ảnh
                st.image(image, caption=f"Prompt {i+1}: {prompt}", use_column_width=True)

                # Tải về theo định dạng
                img_bytes = BytesIO()
                image.save(img_bytes, format=format_option)
                st.download_button(
                    label=f"📥 Tải ảnh {format_option}",
                    data=img_bytes.getvalue(),
                    file_name=f"image_{i+1}.{format_option.lower()}",
                    mime=f"image/{format_option.lower()}"
                )

            except Exception as e:
                st.error(f"Lỗi với prompt {i+1}: {e}")

st.markdown("---")
st.info("Ứng dụng này sử dụng OpenAI DALL·E 3. Hãy đảm bảo bạn có API key hợp lệ.")
