!pip install gradio
!pip install openai
import os
import gradio as gr
from openai import OpenAI




os.environ["NEBIUS_API_KEY"] = "eyJhbGciOiJIUzI1NiIsImtpZCI6IlV6SXJWd1h0dnprLVRvdzlLZWstc0M1akptWXBvX1VaVkxUZlpnMDRlOFUiLCJ0eXAiOiJKV1QifQ.eyJzdWIiOiJnb29nbGUtb2F1dGgyfDExNDAwNjQ4OTM1OTgxOTY5NTUyMyIsInNjb3BlIjoib3BlbmlkIG9mZmxpbmVfYWNjZXNzIiwiaXNzIjoiYXBpX2tleV9pc3N1ZXIiLCJhdWQiOlsiaHR0cHM6Ly9uZWJpdXMtaW5mZXJlbmNlLmV1LmF1dGgwLmNvbS9hcGkvdjIvIl0sImV4cCI6MTkxODIwOTQxMiwidXVpZCI6IjAxOTllN2JiLTdkMGYtNzkxNi05Mzc3LWNiYjdmYzk0MzZkYiIsIm5hbWUiOiJ2aXNobnUiLCJleHBpcmVzX2F0IjoiMjAzMC0xMC0xNFQxMTo1Njo1MiswMDAwIn0._obSvouVH2TwSEQeAnP-pTgH-Ts_yg9vJC9zP0n54gw"


client = OpenAI(
    base_url="https://api.studio.nebius.ai/v1",
    api_key=os.getenv("NEBIUS_API_KEY"),
)


# --- Mock RAG Data Source ---
poet_references = {
    "shakespeare": """Shall I compare thee to a summer’s day?
And summer’s lease hath all too short a date.""",


    "poe": """Once upon a midnight dreary, while I pondered, weak and weary,
Over many a quaint and curious volume of forgotten lore—""",


    "tagore": """Where the mind is without fear and the head is held high,
Where knowledge is free, and words come from the depth of truth."""
}


def get_poetic_context(poet_name):
    """Retrieve poetic style reference for RAG context."""
    return poet_references.get(poet_name.lower(), "")






def generate_poem(emotion, rhyme_scheme, style, theme, language, poet_style):
    """Generate a poem using Nebius AI with optional poet-style context."""


    rhyme_instruction = {
        "abab": (
    "Follow an ABAB rhyme scheme:\n "
    "Line 1 and Line 3 must end with words that rhyme (A).\n "
    "Line 2 and Line 4 must end with words that rhyme (B).\n "
    "Do NOT repeat the exact same end-words; use different words that rhyme.\n"
    "A and B should not rhyme with each other"
),


"aabb": (
    "Follow an AABB rhyme scheme:\n "
    "Line 1 and Line 2 must rhyme with each other (A).\n "
    "Line 3 and Line 4 must rhyme with each other (B).\n "
    "Use different rhyming words instead of repeating the same ones.\n"
    "A and B should not rhyme with each other"
),


"abba": (
    "Follow an ABBA rhyme scheme:\n "
    "Line 1 and Line 4 must rhyme with each other (A).\n "
    "Line 2 and Line 3 must rhyme with each other (B).\n "
    "Make the rhyme clear, but do not repeat the end-words.\n"
    "A and B should not rhyme with each other"
),
        "free verse": "Use free verse with no strict rhyme pattern."
    }.get(rhyme_scheme.lower(), "Use free verse with no strict rhyme pattern.")


    base_prompt = (
        f"Write a 2-stanza poem (each stanza 4 lines) in {language} expressing '{emotion}'. "
        f"The style should be {style}.\nEnsure that it follows {rhyme_instruction} scheme and focus on {theme}."
    )


    if poet_style and poet_style.lower() in poet_references:
        context = get_poetic_context(poet_style)
        base_prompt = (
            f"Using this poetic style as inspiration:\n\n{context}\n\n"
            f"Now, {base_prompt} Ensure it resembles {poet_style.capitalize()}'s tone\n"
            f"Give {rhyme_instruction} the first priority"
        )


    response = client.chat.completions.create(
        model="meta-llama/Meta-Llama-3.1-8B-Instruct",
        temperature=0.85,
        max_tokens=400,
        top_p=0.9,
        messages=[
            {"role": "system", "content": "You are a poetic AI that writes elegant, emotional poems."},
            {"role": "user", "content": base_prompt}
        ]
    )


    return response.choices[0].message.content.strip()




def generate_phrase_poem(phrase, rhyme_scheme, language, style, poet_style):
    """Generate a 2-stanza poem inspired by a user-given phrase."""
    rhyme_instruction = {
        "abab": (
    "Follow an ABAB rhyme scheme:\n "
    "Line 1 and Line 3 must end with words that rhyme (A).\n "
    "Line 2 and Line 4 must end with words that rhyme (B).\n "
    "Do NOT repeat the exact same end-words; use different words that rhyme.\n"
    "A and B should not rhyme with each other"
),


"aabb": (
    "Follow an AABB rhyme scheme:\n "
    "Line 1 and Line 2 must rhyme with each other (A).\n "
    "Line 3 and Line 4 must rhyme with each other (B).\n "
    "Use different rhyming words instead of repeating the same ones.\n"
    "A and B should not rhyme with each other"
),


"abba": (
    "Follow an ABBA rhyme scheme:\n "
    "Line 1 and Line 4 must rhyme with each other (A).\n "
    "Line 2 and Line 3 must rhyme with each other (B).\n "
    "Make the rhyme clear, but do not repeat the end-words.\n"
    "A and B should not rhyme with each other"
),
        "free verse": "Use free verse with no strict rhyme pattern."
    }.get(rhyme_scheme.lower(), "Use free verse with no strict rhyme pattern.")


    base_prompt = (
        f"Write a 2-stanza poem (4 lines each) in {language}, inspired by '{phrase}'. "
        f"The style should be {style}.\nEnsure that it follows {rhyme_instruction} scheme. "
        f"Make it vivid, emotional, and rhythmic."
    )


    if poet_style and poet_style.lower() in poet_references:
        context = get_poetic_context(poet_style)
        base_prompt = (
            f"Using this poetic style as inspiration:\n\n{context}\n\n"
            f"Now, {base_prompt} Ensure it resembles {poet_style.capitalize()}'s tone.\n"
            f"Give {rhyme_instruction} the first priority"
        )


    response = client.chat.completions.create(
        model="meta-llama/Meta-Llama-3.1-8B-Instruct",
        temperature=0.8,
        max_tokens=300,
        messages=[
            {"role": "system", "content": "You are a talented AI poet who crafts inspiring verse."},
            {"role": "user", "content": base_prompt}
        ]
    )


    return response.choices[0].message.content.strip()


def generate_interface(
    choice_val,
    emotion_val, rhyme_scheme_e_val, style_e_val, theme_val, language_e_val, poet_style_e_val,
    phrase_val, rhyme_scheme_p_val, style_p_val, language_p_val, poet_style_p_val
):
    """Handle logic for both emotion-based and phrase-based poem generation."""
    poet_style_e_val = poet_style_e_val if poet_style_e_val != "None" else None
    poet_style_p_val = poet_style_p_val if poet_style_p_val != "None" else None


    if choice_val == "Emotion-based":
        return generate_poem(
            emotion=emotion_val,
            rhyme_scheme=rhyme_scheme_e_val,
            style=style_e_val,
            theme=theme_val,
            language=language_e_val,
            poet_style=poet_style_e_val
        )
    else:
        return generate_phrase_poem(
            phrase=phrase_val,
            rhyme_scheme=rhyme_scheme_p_val,
            language=language_p_val,
            style=style_p_val,
            poet_style=poet_style_p_val
        )






with gr.Blocks(title="🎭 AI Poem Generator (Nebius + RAG)") as demo:
    gr.Markdown("# 🎭 AI Poem Generator (Nebius AI + RAG)")
    gr.Markdown("Generate creative poems based on *emotions* or *phrases*, with optional poet-inspired styles!")


    choice = gr.Radio(
        ["Emotion-based", "Phrase-based"],
        label="Choose Poem Type",
        value="Emotion-based"
    )


    with gr.Group(visible=True) as emotion_inputs:
        emotion = gr.Textbox(label="Emotion", placeholder="love, joy, anger, hope")
        rhyme_scheme = gr.Dropdown(["ABAB", "AABB", "ABBA", "free verse"], label="Rhyme Scheme", value="free verse")
        style = gr.Textbox(label="Style", placeholder="classical, modern, haiku, lyrical")
        theme = gr.Textbox(label="Theme", placeholder="nature, dreams, friendship")
        language = gr.Textbox(label="Language", value="English")
        poet_style = gr.Dropdown(["None", "Shakespeare", "Poe", "Tagore"], label="Poet Style", value="None")


    with gr.Group(visible=False) as phrase_inputs:
        phrase = gr.Textbox(label="Phrase", placeholder="e.g., The stars whisper secrets")
        rhyme_scheme_p = gr.Dropdown(["ABAB", "AABB", "ABBA", "free verse"], label="Rhyme Scheme", value="free verse")
        style_p = gr.Textbox(label="Style", placeholder="lyrical, modern")
        language_p = gr.Textbox(label="Language", value="English")
        poet_style_p = gr.Dropdown(["None", "Shakespeare", "Poe", "Tagore"], label="Poet Style", value="None")


    output = gr.Textbox(label="✨ Generated Poem ✨", lines=10)


    def toggle(choice):
        return (
            gr.update(visible=choice == "Emotion-based"),
            gr.update(visible=choice == "Phrase-based")
        )


    choice.change(toggle, choice, [emotion_inputs, phrase_inputs])


    generate_btn = gr.Button("✨ Generate Poem ✨")


    generate_btn.click(
        fn=generate_interface,
        inputs=[
            choice,
            emotion,
            rhyme_scheme,
            style,
            theme,
            language,
            poet_style,
            phrase,
            rhyme_scheme_p,
            style_p,
            language_p,
            poet_style_p
        ],
        outputs=output
    )
demo.launch(share=True)
