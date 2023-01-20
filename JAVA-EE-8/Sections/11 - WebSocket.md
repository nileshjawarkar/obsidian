- Payload
``` java
package com.nilesh.jawarkar.learn.javaee8.chat;

public class Message {
	private final String author;
	private final String content;
	
	public Message(final String author, final String content) {
		this.author = author;
		this.content = content;
	}
	
	public String getAuthor() {
		return this.author;
	}
	
	public String getContent() {
		return this.content;
	}
}
```

- Decoder
``` java
package com.nilesh.jawarkar.learn.javaee8.chat;

import java.io.StringReader;
import javax.json.Json;
import javax.json.JsonObject;
import javax.websocket.DecodeException;
import javax.websocket.Decoder;
import javax.websocket.EndpointConfig;

public class MessageDecoder implements Decoder.Text<Message> {
	@Override
	public void destroy() {
	}

	@Override
	public void init(final EndpointConfig config) {
	}

	@Override
	public Message decode(final String jsonString) throws DecodeException {
		JsonObject obj = Json.createReader(
			new StringReader(jsonString)).readObject();
		return new Message(obj.getString("author"), obj.getString("content"));
	}

	@Override
	public boolean willDecode(final String arg0) {
		return true;
	}
}
```

- Encoder
``` java
package com.nilesh.jawarkar.learn.javaee8.chat;

import javax.json.Json;
import javax.websocket.EncodeException;
import javax.websocket.Encoder;
import javax.websocket.EndpointConfig;

public class MessageEncoder implements Encoder.Text<Message> {
	@Override
	public void destroy() {
	}

	@Override
	public void init(final EndpointConfig arg0) {
	}

	@Override
	public String encode(final Message message) throws EncodeException {
		return Json.createObjectBuilder().add("author", message.getAuthor())
			.add("content", message.getContent()).build().toString();
	}
}
```

- Server
``` java
package com.nilesh.jawarkar.learn.javaee8.chat;

import java.io.IOException;
import javax.websocket.EncodeException;
import javax.websocket.OnMessage;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint(value = "/chat", encoders = MessageEncoder.class, decoders = MessageDecoder.class)
public class MessageServer {

	@OnMessage
	public void onMessgae(final Message message, final Session session) {
		// -- Prepare response
		Message res = buildResponse(message);
		try {
			session.getBasicRemote().sendObject(res);
		} catch (IOException e) {
			e.printStackTrace();
		} catch (EncodeException e) {
			e.printStackTrace();
		}
	}

	private Message buildResponse(final Message message) {
		StringBuilder resBuilder = new StringBuilder().append("Hi ")
				.append(message.getAuthor()).append(" - ");
		if (message.getContent().contains("ping")) {
			resBuilder.append("PONG");
		}
		return new Message("System", resBuilder.toString());
	}
}
```

- Client

``` java
package com.nilesh.jawarkar.learn.javaee8.chat;

import java.io.IOException;
import javax.websocket.ClientEndpoint;
import javax.websocket.CloseReason;
import javax.websocket.EncodeException;
import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;

@ClientEndpoint(encoders = MessageEncoder.class, decoders = MessageDecoder.class)
public class MessageClient {
	private Session userSession = null;

	@OnOpen
	public void onOpen(final Session userSession) {
		System.out.println("opening websocket");
		this.userSession = userSession;
	}

	@OnClose
	public void onClose(final Session userSession, final CloseReason reason) {
		System.out.println("closing websocket");
		this.userSession = null;
	}

	@OnMessage
	public void onMessage(final Message message, final Session session) {
		System.out.println("Message - " + message.getContent());
	}

	public void send(final Message message) {
		if (this.userSession != null) {
			try {
				this.userSession.getBasicRemote().sendObject(message);
			} catch (IOException | EncodeException e) {
				e.printStackTrace();
			}
		}
	}
}
```

``` java
package com.nilesh.jawarkar.learn.javaee8.chat;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URL;
import java.util.concurrent.locks.LockSupport;

import javax.websocket.ContainerProvider;
import javax.websocket.DeploymentException;
import javax.websocket.WebSocketContainer;

import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.arquillian.test.api.ArquillianResource;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(Arquillian.class)
public class TestMessageClient {
	@Deployment
	public static WebArchive createDeployment() {
		return ShrinkWrap.create(WebArchive.class, "carman.war")
				.addPackage("com.nilesh.jawarkar.learn.javaee8.chat")
				.addAsWebInfResource("beans.xml", "beans.xml");
	}

	@ArquillianResource
	private URL url;
	private String strURL = null;

	@After
	public void cleanUp() {
	}

	@Before
	public void init() {
		this.strURL = "ws://localhost:8080/carman/chat";
		final String startPort = System.getProperty("tomee.httpPort");
		if (this.url != null) {
			this.strURL = this.url.toString().replace("http:", "ws:") + "/chat";
		} else if (startPort != null) {
			this.strURL = "ws://localhost:" + startPort + "/carman/chat";
		}
		System.out.println("URL = " + this.strURL);
	}

	@Test
	public void testClientAndServer() {
		MessageClient client = new MessageClient();
		WebSocketContainer container = ContainerProvider.getWebSocketContainer();
		try {
			container.connectToServer(client, new URI(this.strURL));
			Message msg1 = new Message("Nilesh", "This is test ping.");
			Message msg2 = new Message("Pankaj", "This is test ping.");
			Message msg3 = new Message("Gaurav", "This is test ping.");

			System.out.println("Sending message from - " + msg1.getAuthor() 
				+ ", msg = " + msg1.getContent());
			client.send(msg1);

			System.out.println("Sending message from - " + msg2.getAuthor() 
				+ ", msg = " + msg2.getContent());
			client.send(msg2);

			System.out.println("Sending message from - " + msg3.getAuthor() 
				+ ", msg = " + msg3.getContent());
			client.send(msg3);
			LockSupport.parkNanos(2000000000L);
		} catch (DeploymentException | IOException | URISyntaxException e) {
			e.printStackTrace();
		}
	}
}
```