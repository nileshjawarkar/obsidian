
Server-Sent Events (SSE) is an HTTP based specification that provides a way to establish a long-running and mono-channel connection from the server to the client. The client initiates the SSE connection by using the media type _text/event-stream_ in the _Accept_ header. Later, it gets updated automatically without requesting the server.


``` java
package com.nilesh.jawarkar.learn.javaee8.boundry;

import javax.annotation.PostConstruct;
import javax.ejb.Singleton;
import javax.enterprise.event.Observes;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.sse.Sse;
import javax.ws.rs.sse.SseBroadcaster;
import javax.ws.rs.sse.SseEventSink;

import com.nilesh.jawarkar.learn.javaee8.entity.CarCreated;

@Path("car-created-sse")
@Singleton // -- Multiple instance of this make no sense.
public class CarCreatedSSE {
	// -- 1) inject sse context
	// -- Injection not working in tomee
	@Context
	Sse sse;
	SseBroadcaster broadcaster = null;

	// -- 2) create broadcaster
	@PostConstruct
	public void init() {
		if (this.sse != null) {
			this.broadcaster = this.sse.newBroadcaster();
		}
	}

	// -- 4) Broadcast events
	public void onCarCreation(@Observes final CarCreated carCreated) {
		if (this.broadcaster != null) {
			this.broadcaster.broadcast(
				this.sse.newEvent(carCreated.getIdentifier()));
		}
	}

	// -- 3) Register client for broadcast
	@GET
	@Produces(MediaType.SERVER_SENT_EVENTS)
	public void registerForEvent(@Context final Sse sse,
			@Context final SseEventSink clientEventSink) {
		if (this.broadcaster == null) {
			this.sse = sse;
			this.broadcaster = sse.newBroadcaster();
		}
		this.broadcaster.register(clientEventSink);
	}
}
```

- Client
``` java
package com.nilesh.jawarkar.learn.javaee8.boundry;

import static org.junit.Assert.assertEquals;
import java.net.URL;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;
import javax.annotation.Resource;
import javax.enterprise.concurrent.ManagedScheduledExecutorService;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.Entity;
import javax.ws.rs.client.WebTarget;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.sse.SseEventSource;
import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.arquillian.test.api.ArquillianResource;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

@RunWith(Arquillian.class)
public class TestCarCreatedEventBroadcast {
	@Deployment
	public static WebArchive createDeployment() {
		return ShrinkWrap.create(WebArchive.class, "carman.war")
		        .addPackage("com.nilesh.jawarkar.learn.javaee8.boundry")
		        .addPackage("com.nilesh.jawarkar.learn.javaee8.config")
		        .addPackage("com.nilesh.jawarkar.learn.javaee8.control")
		        .addPackage("com.nilesh.jawarkar.learn.javaee8.entity")
		        .addAsResource("persistence.xml", "META-INF/persistence.xml")
		        .addAsWebInfResource("beans.xml", "beans.xml");
	}

	@ArquillianResource
	private URL                     url;
	private WebTarget               sseTarget;
	private WebTarget               createTarget;
	private Client                  client;
	private int                     eventRecieved = 0;

	@Resource(name = "DefaultManagedExecutorService")
	ManagedScheduledExecutorService mses;

	@After
	public void clear() {
		client.close();
	}

	public void createCar() {
		createCar(Color.BLUE, EngineType.DIESEL);
		createCar(Color.RED, EngineType.PETROL);
		createCar(Color.WHITE, EngineType.PETROL);
		createCar(Color.BLUE, EngineType.ELECTRIC);
	}

	private void createCar(final Color color, final EngineType engineType) {
		final Specification spec = new Specification();
		spec.setColor(color);
		spec.setEngineType(engineType);
		createTarget.request(MediaType.APPLICATION_JSON).post(Entity.json(spec));
	}

	@Before
	public void init() {
		String       strURL    = "http://localhost:8080/carman/resources";
		final String startPort = System.getProperty("tomee.httpPort");
		if (url != null) {
			strURL = url.toString() + "/resources";
		} else if (startPort != null) {
			strURL = "http://localhost:" + startPort + "/carman/resources";
		}
		System.out.println("URL = " + strURL);
		client       = ClientBuilder.newClient();
		sseTarget    = client.target(strURL + "/car-created-sse");
		createTarget = client.target(strURL + "/cars");
	}

	@Test
	public void testCarCreatedSSE() {
		final SseEventSource source = SseEventSource.target(sseTarget).build();
		source.register(event -> {
			System.out.println(">>> " + event.readData());
			eventRecieved += 1;
		});
		mses.schedule(() -> createCar(), 1, TimeUnit.SECONDS);

		source.open();
		LockSupport.parkNanos(2000000000L);
		source.close();
		assertEquals(4, eventRecieved);
	}
}
```