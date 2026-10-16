---
title: "[furiosa-opt v0.8.1] kernel launch 시 image upload 비용 관련 문의"
url: "https://forums.furiosa.ai/t/furiosa-opt-v0-8-1-kernel-launch-image-upload/474#post_2"
date: "2026-10-08"
author: "@yan Youseok Yang"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
안녕하세요, 각 질문에 먼저 답 드리겠습니다. Kernel은 load시 NPU HBM에 올라가 있는데, HBM에서 instruction memory의 copy가 inline으로 발생하고 있습니다. 이 부분은 implementation detail로써 내부적으로 구현이 되어야 하는데, 아직 반영이 안된것에 가깝습니다.
