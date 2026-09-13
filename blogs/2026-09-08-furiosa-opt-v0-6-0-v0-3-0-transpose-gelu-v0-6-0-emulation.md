---
title: "[furiosa-opt v0.6.0] v0.3.0 에서 동작하던 transpose/gelu 커널이 v0.6.0 emulation에서 실패 - 권장 패턴 문의"
url: "https://forums.furiosa.ai/t/furiosa-opt-v0-6-0-v0-3-0-transpose-gelu-v0-6-0-emulation/458#post_1"
date: "2026-09-08"
author: "@mskim92 김민수"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
안녕하세요. furiosa-opt-std v0.3.0 API로 작성한 커널 코드를 v0.6.0 API를 반영하여 수정 및 보완하고 있습니다. 이전 버전에서 테스트 진행했던 대부분 커널(contraction, elementwise 등)은 API 변경 사항(to_hbm 시그니처 변경, contract_outer Acc 추가 등)에 맞춰 수정한 후 v0.6.0 emulation 백엔드에서 정상 동작하는 것을 확인했습니다.
