---
title: "[furiosa-opt v0.6.0]같은 slice의 DM 텐서를 다른 축 조합으로 commit_view할 때 StreamUnmatchedSegment가 발생합니다"
url: "https://forums.furiosa.ai/t/furiosa-opt-v0-6-0-slice-dm-commit-view-streamunmatchedsegment/453#post_2"
date: "2026-08-19"
author: "@jeongmin.park Jeongmin Park"
feed_url: "https://forums.furiosa.ai/posts.rss"
---
안녕하세요, commit engine의 경우에, commit_trim으로 생성된 stream mapping과, write 대상의 Element mapping을 비교했을 때: stream Packet과 Element의 innermost Packet::SIZE 만큼이 완전히 동일해야합니다. 해당 예제에서, commit_trim의 output Packet: m![COLUMNS % 32], Element: m![ROWS % 32, COLUMNS % 16 % 2, DIGIT_SLOT = 1 #{!} 8, COLUMNS / 16] 에서, Element % Packet::SIZE(=32) 는 [DIGIT_SLOT = 1 #{!} 2, COLUMNS / 16] 인데, 이것이 commit_trim의 output Packet과 다르기 때문에 commit engine에서 받아들일 수 없는 형태에 해당합니다. 이러한 형태를 컴파일 하기 위해서는, transpose engine을 이용하여 Element innermost와 commit_trim output Packet을 맞추는 방향으로 커널을 작성해야 합니다.
