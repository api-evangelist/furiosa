---
title: "Furiosa-LLM 프로파일링 방법"
url: "https://forums.furiosa.ai/t/furiosa-llm/364#post_8"
date: "2026-09-05"
author: "@hyunsik Hyunsik Choi"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
안녕하세요? 물론입니다. 다음과 같은 파일을 작성 하신 뒤에, from furiosa_llm import LLM, SamplingParams with LLM("furiosa-ai/Qwen2.5-0.5B-Instruct") as llm: messages = [ {"role": "user", "content": "What is the capital of France?"} ] prompt = llm.tokenizer.apply_chat_template( messages, tokenize=False, add_generation_prompt=True, ) outputs = llm.generate( [prompt], SamplingParams(temperature=0.0, max_tokens=16), ) print(outputs[0].outputs[0].text) # with 블록을 벗어나면 profiler가 종료되면서 profile.json이 생성 다음과 같이 실행하시면 됩니다.
