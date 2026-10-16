---
title: "[furiosa-opt v0.8.1] 제출 빌드가 \"the build exceeded the build server's memory limit\"로 실패합니다 (한도와 병렬도 문의)"
url: "https://forums.furiosa.ai/t/furiosa-opt-v0-8-1-the-build-exceeded-the-build-servers-memory-limit/481#post_1"
date: "2026-10-10"
author: "@hyeb1717 Hyebeen Seo"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
안녕하세요. 환경 furiosa-opt-std 0.8.1, moa-submitter 2.1.0 Stage 2 제출(moa-submitter submit --concurrency 8) 현상 커널을 몇 개 더한 트리를 제출하면, 빌드 단계 약 3분째에 the build exceeded the build server’s memory limit로 실패합니다. 같은 트리를 다시 내도 같았습니다(일시적인 문제는 아닌 것 같습니다).
