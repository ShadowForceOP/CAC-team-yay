var express = require('express');
var app = express();
var serv = require('http').Server(app);

app.use('/', express.static(__dirname + '/client/'));

serv.listen(2000);
console.log('Server started.');

var chats = [];
var socketList = {};
var guests = 0;
var io = require('socket.io')(serv, {});
var fs = require('fs');
var d = new Date();

io.sockets.on('connection', socket => {
  if(!chats.length)
	  getChats();
	emitChats();
	socket.color = getRandomColor();

	socket.on('name', name => {
	  refreshTime();
	  name = encode(name);
		socketList[socket.id] = socket;
		if (name == 'Guest') {
			socket.name = name + ' ' + guests;
			guests++;
		} else socket.name = name;
		chats.push({
		  message:
			'<span class="name" style="color:' +
				socket.color +
				'">' +
				socket.name +
				'</span><span class="green"> joined.</span>',
				time: d.getTime()
		});
		emitChats();
		emitWho();
		saveChats();
	});

	socket.on('message', message => {
	  refreshTime();
	  if(message.length > 100) return;
	  message = encode(message);
		chats.push({
		  message:
			'<span class="name" style="color:' +
				socket.color +
				'">' +
				socket.name +
				'</span>: <span class="message-content">' +
				message + '</span>',
				time: d.getTime(),
				socketId: socket.id,
				id: Math.random()
		});
		emitChats();
		saveChats();
	});
  socket.on('delete', id => {
    for(var i in chats){
      if(chats[i].id == id){
        chats.splice(i, 1);
        break;
      }
    }
    emitChats();
  });
	socket.on('disconnect', () => {
	  refreshTime();
		if (socketList[socket.id]) {
			chats.push({
			  message:
				'<span class="name" style="color:' +
					socket.color +
					'">' +
					socket.name +
					'</span> <span class="red"> left.</span>',
				time: d.getTime()
			});
			emitChats();
			delete socketList[socket.id];
		}
		saveChats();
		emitWho();
	});
});
setInterval(() => {
  refreshTime();
  emitChats();
}, 10000);
function refreshTime(){
  d = new Date();
  io.sockets.emit('time', d.getTime());
}
function emitWho() {
	for (var j in socketList) {
		var pack = [];
		for (var i in socketList) {
			if (j == i) {
				pack.push({
					name: socketList[i].name + ' (You)',
					color: socketList[i].color
				});
			} else {
				pack.push({ name: socketList[i].name, color: socketList[i].color });
			}
		}
		socketList[j].emit('who', pack);
	}
}
function emitChats(){
 for(var i in socketList){
    var pack = chats.map(c => {
      if(c.socketId != i) {
        return {message: c.message, time: c.time};
      }
      else{
        return c;
      }
    });
    socketList[i].emit('chats', pack);
  }
}
function saveChats() {
  var chatsStr = chats.map(e => e.message + "\n" + e.time).join("\n\n");
	fs.writeFile('chats.txt', chatsStr, function(err) {
		if (err) {
			return console.log(err);
		}
	});
}

function getChats() {
	fs.readFile('chats.txt', 'utf8', function(err, data) {
		if (err) {
			return console.log(err);
		}
		if (data.length) 
		  chats = data.split('\n\n').map(e => 
		    ({message: e.split("\n")[0], time: e.split("\n")[1]})
		  );
	});
}
function encode(txt) {
	return txt.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
}

function getRandomColor() {
	var letters = '0123456789ABCDEF';
	var color = '#';
	for (var i = 0; i < 6; i++) {
		color += letters[Math.floor(Math.random() * 14)];
	}
	return color;
}
