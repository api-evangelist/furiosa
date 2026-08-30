---
title: "[furiosa-opt v0.5.1] dma_gather_unscaled의 64 KiB SPM 요청에서 V1 scheduler 실패"
url: "https://forums.furiosa.ai/t/furiosa-opt-v0-5-1-dma-gather-unscaled-64-kib-spm-v1-scheduler/450#post_2"
date: "2026-08-12"
author: "@dohyun Dohyun Kim"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
안녕하세요. 지금 형태처럼 큰 index tensor에 unscaled gather를 쓰는 것은 SPM 크기 제약 때문에 컴파일 불가능한게 맞습니다. 권장하는 방법은 scaled gather이고, unscaled gather를 유지해야 한다면 index를 SPM에 들어가는 크기로 쪼개서 여러 번 실행하셔야 합니다.
