---
title: "[furiosa-opt v0.6.0] AllReduce 관련 문의"
url: "https://forums.furiosa.ai/t/furiosa-opt-v0-6-0-allreduce/467#post_1"
date: "2026-09-20"
author: "@curling_grad Ryang Sohn"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
안녕하세요, MICRO 2026 MOA 대회 환경인 0.6.0 버전에 맞춰 커널을 작성 중인 상황입니다. 여러 cluster를 사용하기 위해 furiosa-opt book에 나온 AllReduce 예시 와 비슷한 코드를 작성하려고 하는데요, cluster_swap 함수가 구현되지 않아 찾아보니 0.6.0에서는 dm_cluster_shuffle 함수를 대신 사용해야 하는 것으로 보입니다. 그런데 NPU 타깃으로 컴파일을 시도했을 때 TensorDmaClusterSwap lowering is not yet implemented 에러가 발생하면서 진행이 되지 않습니다.
