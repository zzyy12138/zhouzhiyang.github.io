---
layout: post
title: "Python Web开发-WebSocket实时通信"
date: 2019-03-30 
description: "Flask-SocketIO、实时消息、房间管理"
tag: Python 

---

### Flask-SocketIO

>```python
>from flask_socketio import SocketIO, emit, join_room, leave_room
>
>@socketio.on('connect')
>def on_connect():
>    emit('status', {'msg': 'Connected'})
>
>@socketio.on('join')
>def on_join(data):
>    room = data['room']
>    join_room(room)
>    emit('status', {'msg': f'Joined room {room}'})
>```

### 房间管理

>```python
>@socketio.on('message')
>def handle_message(data):
>    room = data['room']
>    emit('message', data, room=room)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-WebSocket实时通信](http://zhouzhiyang.cn/2019/03/Python_Web_WebSocket/) 

