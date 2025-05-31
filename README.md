# TranscriWeb
transcrição e tradução de vídeos do YouTube, com um toque a mais.
import streamlit as st
from youtube_transcript_api import YouTubeTranscriptApi
from deepmultilingualpunctuation import PunctuationModel
from deep_translator import GoogleTranslator
import re

# Função para extrair o ID do vídeo a partir do link
def get_video_id(url):
    match = re.search(r"(?:v=|youtu\.be/)([a-zA-Z0-9_-]{11})", url)
    return match.group(1) if match else None

# Função para baixar a transcrição do vídeo
def get_transcript(video_id):
    try:
        transcript = YouTubeTranscriptApi.get_transcript(video_id, languages=['pt', 'pt-BR', 'en'])
        full_text = ' '.join([entry['text'] for entry in transcript])
        return full_text
    except Exception as e:
        return None

# Função para pontuar e quebrar em parágrafos
def format_text(text):
    model = PunctuationModel()
    punctuated = model.restore_punctuation(text)
    # Quebra em parágrafos com base na pontuação
    paragraphs = re.split(r'(?<=[.!?])\s+', punctuated)
    return '\n\n'.join(paragraphs)

# Função para traduzir o texto
def translate_text(text, target_lang):
    try:
        translated = GoogleTranslator(source='auto', target=target_lang).translate(text)
        return translated
    except Exception as e:
        return None

# Interface do app
st.title("🌐 Transcritor, Organizador e Tradutor de Vídeos do YouTube")

url = st.text_input("📺 Cole o link do vídeo do YouTube:")

# Seletor de idioma
languages = {
    "Português": "pt",
    "Inglês": "en",
    "Espanhol": "es",
    "Francês": "fr",
    "Alemão": "de",
    "Italiano": "it",
    "Russo": "ru"
}
selected_lang = st.selectbox("🌍 Traduzir para:", list(languages.keys()))

if url:
    with st.spinner("🔍 Processando transcrição..."):
        video_id = get_video_id(url)
        if not video_id:
            st.error("❌ Link inválido.")
        else:
            raw_text = get_transcript(video_id)
            if raw_text:
                formatted = format_text(raw_text)

                # Tradução
                translated = translate_text(formatted, languages[selected_lang])
                if translated:
                    st.success(f"✅ Transcrição traduzida para {selected_lang} com sucesso!")
                    st.text_area("📄 Texto traduzido e formatado:", translated, height=500)

                    # Botão para download
                    file_name = f"transcricao_{video_id}_{languages[selected_lang]}.txt"
                    st.download_button(
                        label="📥 Baixar como .TXT",
                        data=translated,
                        file_name=file_name,
                        mime="text/plain"
                    )
                else:
                    st.error("❌ Ocorreu um erro na tradução.")
            else:
                st.error("❌ Não foi possível obter a transcrição. O vídeo pode não ter legendas disponíveis.")
