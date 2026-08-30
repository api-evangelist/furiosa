---
title: "[furiosa-opt v0.5.1]8 KiB/slice VRF operand의 연속 vector pipeline scheduling 실패"
url: "https://forums.furiosa.ai/t/furiosa-opt-v0-5-1-8-kib-slice-vrf-operand-vector-pipeline-scheduling/451#post_4"
date: "2026-08-14"
author: "@chaehyun.jeong Chaehyun Jeong"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
확인했습니다. 지금 보니 문서에 설명이 빠져있는데, vector_stash() 는 현재 running tensor 의 일부를 vrf 에 저장해두기 때문에 (현재는 보수적으로 1024B 로 되어 있습니다) 이 용량을 고려해야 합니다. 이미 modulus_vrf 텐서가 VRF에 8KiB 를 꽉 채워쓰는데, vector_stash를 불렀기 때문에 용량 초과 에러가 난 것으로 보입니다.
