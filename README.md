# jarvis-from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
from openai import OpenAI
import os
from pptx import Presentation
from pptx.util import Inches
import requests
from datetime import datetime

app = FastAPI(title="JARVIS Cloud Brain - Rafael")

# CONFIGURA AQUÍ TU API KEY (Groq es gratis y usa Llama 3.1 como yo)
client = OpenAI(
    api_key=os.getenv("GROQ_API_KEY"),
    base_url="https://api.groq.com/openai/v1"
)

SYSTEM = """
Eres JARVIS, el asistente de Rafael Valentin Espinoza Gil.
Tono: Leal, inteligente, eficiente, en español.
Misión: Hacer investigaciones profundas y crear presentaciones profesionales.
Siempre responde conciso y di que estás haciendo.
"""

class Task(BaseModel):
    texto: str
    origen: str = "celular"

def buscar_web(query: str):
    # Usa Tavily o Serper - gratis para investigaciones
    # Aquí placeholder, tu servidor lo hará con tu API KEY
    return f"Resultados de investigación sobre: {query}"

@app.post("/jarvis")
async def jarvis(task: Task):
    # 1. Decide qué hacer
    prompt = f"{SYSTEM}\nUsuario ({task.origen}): {task.texto}\n¿Es investigación o presentación? Si es presentación, dame el título y los 5 puntos clave en JSON."
    
    completion = client.chat.completions.create(
        model="llama-3.1-70b-versatile",
        messages=[{"role": "user", "content": prompt}]
    )
    respuesta_texto = completion.choices[0].message.content

    # 2. Si detecta que quiere presentación, la crea
    if "presentacion" in task.texto.lower() or "diapositivas" in task.texto.lower():
        prs = Presentation()
        prs.slide_width = Inches(13.33)
        prs.slide_height = Inches(7.5)
        
        # Slide 1 - Portada
        slide = prs.slides.add_slide(prs.slide_layouts[6])
        slide.background.fill.solid()
        slide.background.fill.fore_color.rgb = slide.background.fill.fore_color.rgb.from_string("0A0A23")
        # (Aquí va tu diseño tipo Stark)

        filename = f"/tmp/presentacion_{datetime.now().strftime('%Y%m%d_%H%M')}.pptx"
        prs.save(filename)
        return {"respuesta": f"Listo Rafael. Creé tu presentación. {respuesta_texto}", "archivo": filename}
    
    return {"respuesta": respuesta_texto}

@app.get("/")
def home():
    return {"status": "JARVIS ONLINE - Esperando órdenes de Rafael"}cerebro

fastapi
uvicorn
openai
python-pptx
requests
python-multipart
