---
layout: post
title: Handling SIGQUIT to gracefully shutdown Starman
tags:
- perl
---
[Handling SIGQUIT to gracefully shutdown
Starman](https://github.com/miyagawa/Starman/compare/quit-graceful-shutdown)

Implemented QUIT handler to gracefully shutdown Starman workers (and full
graceful restart with Server::Starter) - one of the ugliest monkeypatch I've
written in a few months.

HUPのリスタート処理がlinux環境で app.psgi 以外のファイルパスを使っていた場合にうまく動いていなかったのを直したのと同時に、Server::
Starter対応の部分で、デフォルトでTERMを受け取ったらgraceful
shutdownしないといけないのが、そこが実装されてなかったのに気づいた。というわけで、[SIGQUITでgraceful shutdown](https://github.com/miyagawa/Starman/commit/109ed98eeac84224db9b09777cff66cb68f3d007
) するようにパッチを書いたので、`start_server --signal-on-hup=QUIT -- starman --preload-app
...` で graceful restart 可能になった。

しかし、Net::Server のシグナル処理は非常に拡張しづらいので、register_sig をlocalで上書きしてとかmonkey
patchしまくり。かといって今更Net::Serverにパッチをあてるのもどうかという気もするし（2010年10月に0.99が出て以来更新されていない）

