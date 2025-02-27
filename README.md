# Organisation Flapdoodle OSS
[![Build Status](https://drone.io/github.com/flapdoodle-oss/embedmongo.flapdoodle.de/status.png)](https://drone.io/github.com/flapdoodle-oss/embedmongo.flapdoodle.de/latest)

We are now a github organisation. You are invited to participate. :)

# Embedded MongoDB

Embedded MongoDB will provide a platform neutral way for running mongodb in unittests.

## Why?

- dropping databases causing some pains (often you have to wait long time after each test)
- its easy, much easier as installing right version by hand
- you can change version per test

## Dependencies

### Build on top of

- Embed Process Util [de.flapdoodle.embed.process](https://github.com/flapdoodle-oss/de.flapdoodle.embed.process)

### Other ways to use Embedded MongoDB

- in a Maven build using [embedmongo-maven-plugin](https://github.com/joelittlejohn/embedmongo-maven-plugin)
- in a Clojure/Leiningen project using [lein-embongo](https://github.com/joelittlejohn/lein-embongo)
- in a Scala/specs2 specification using [specs2-embedmongo](https://github.com/athieriot/specs2-embedmongo)
- in Scala tests using [scalatest-embedmongo](https://github.com/SimplyScala/scalatest-embedmongo)

### Comments about Embedded MongoDB in the Wild

- http://stackoverflow.com/questions/6437226/embedded-mongodb-when-running-integration-tests
- http://www.cubeia.com/index.php/blog/archives/436
- http://blog.diabol.se/?p=390

### Other MongoDB Stuff

- https://github.com/thiloplanz/jmockmongo - mongodb mocking
- https://github.com/lordofthejars/nosql-unit - extended nosql unit testing
- https://github.com/jirutka/embedmongo-spring - Spring Factory Bean for EmbedMongo

## Howto

### Maven

**IMPORTANT NOTE: maven groupId and artifactId change**

*	groupId from __de.flapdoodle.embedmongo__ to __de.flapdoodle.embed__
*	artifactId from __de.flapdoodle.embedmongo__ to __de.flapdoodle.embed.mongo__

Stable (Maven Central Repository, Released: 05.09.2013 - wait 24hrs for [maven central](http://repo1.maven.org/maven2/de/flapdoodle/embed/de.flapdoodle.embed.mongo/maven-metadata.xml))

	<dependency>
		<groupId>de.flapdoodle.embed</groupId>
		<artifactId>de.flapdoodle.embed.mongo</artifactId>
		<version>1.36</version>
	</dependency>

Snapshots (Repository http://oss.sonatype.org/content/repositories/snapshots)

	<dependency>
		<groupId>de.flapdoodle.embed</groupId>
		<artifactId>de.flapdoodle.embed.mongo</artifactId>
		<version>1.37-SNAPSHOT</version>
	</dependency>


### Build from source

When you fork or clone our branch you should always be able to build the library by running 

	mvn package

There is also a build.gradle file available which might sometimes be outdated but we try to keep it working. So the gradle command is

	gradle build

Or if you want to use the gradle wrapper:

	./gradlew build
 
### Changelog

[Changelog](Changelog.md)

### Supported Versions

Versions: some older, a stable and a development version
Support for Linux, Windows and MacOSX.

### Usage
	import de.flapdoodle.embed.mongo.config.ArtifactStoreBuilder;
	
	...
	
	int port = 12345;
	IMongodConfig mongodConfig = new MongodConfigBuilder()
		.version(Version.Main.PRODUCTION)
		.net(new Net(port, Network.localhostIsIPv6()))
		.build();

	MongodStarter runtime = MongodStarter.getDefaultInstance();

	MongodExecutable mongodExecutable = null;
	try {
		mongodExecutable = runtime.prepare(mongodConfig);
		MongodProcess mongod = mongodExecutable.start();

		MongoClient mongo = new MongoClient("localhost", port);
		DB db = mongo.getDB("test");
		DBCollection col = db.createCollection("testCol", new BasicDBObject());
		col.save(new BasicDBObject("testDoc", new Date()));

	} finally {
		if (mongodExecutable != null)
			mongodExecutable.stop();
	}

### Usage - custom mongod filename 
	import de.flapdoodle.embed.mongo.config.ArtifactStoreBuilder;
	
	...

	int port = 12345;
	IMongodConfig mongodConfig = new MongodConfigBuilder()
		.version(Version.Main.PRODUCTION)
		.net(new Net(port, Network.localhostIsIPv6()))
		.build();

	Command command = Command.MongoD;

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaults(command)
		.artifactStore(new ArtifactStoreBuilder()
			.defaults(command)
			.download(new DownloadConfigBuilder()
				.defaultsForCommand(command))
				.executableNaming(new UserTempNaming()))
		.build();

	MongodStarter runtime = MongodStarter.getInstance(runtimeConfig);

	MongodExecutable mongodExecutable = null;
	try {
		mongodExecutable = runtime.prepare(mongodConfig);
		MongodProcess mongod = mongodExecutable.start();

		MongoClient mongo = new MongoClient("localhost", port);
		DB db = mongo.getDB("test");
		DBCollection col = db.createCollection("testCol", new BasicDBObject());
		col.save(new BasicDBObject("testDoc", new Date()));

	} finally {
		if (mongodExecutable != null)
			mongodExecutable.stop();
	}

### Unit Tests

	public abstract class AbstractMongoDBTest extends TestCase {

		private MongodExecutable _mongodExe;
		private MongodProcess _mongod;

		private MongoClient _mongo;
		@Override
		protected void setUp() throws Exception {

			MongodStarter runtime = MongodStarter.getDefaultInstance();
			_mongodExe = runtime.prepare(new MongodConfigBuilder()
				.version(Version.Main.PRODUCTION)
				.net(new Net(12345, Network.localhostIsIPv6()))
				.build());
			_mongod = _mongodExe.start();

			super.setUp();

			_mongo = new MongoClient("localhost", 12345);
		}

		@Override
		protected void tearDown() throws Exception {
			super.tearDown();

			_mongod.stop();
			_mongodExe.stop();
		}

		public Mongo getMongo() {
			return _mongo;
		}

	}

#### ... with some more help

	...
	MongodForTestsFactory factory = null;
	try {
		factory = MongodForTestsFactory.with(Version.Main.PRODUCTION);

		MongoClient mongo = factory.newMongo();
		DB db = mongo.getDB("test-" + UUID.randomUUID());
		DBCollection col = db.createCollection("testCol", new BasicDBObject());
		col.save(new BasicDBObject("testDoc", new Date()));

	} finally {
		if (factory != null)
			factory.shutdown();
	}
	...

### Customize Download URL

	...
	Command command = Command.MongoD;

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaults(command)
		.artifactStore(new ArtifactStoreBuilder()
			.defaults(command)
			.download(new DownloadConfigBuilder()
				.defaultsForCommand(command)
				.downloadPath("http://my.custom.download.domain/")))
		.build();
	...

### Customize Artifact Storage

	...
	IDirectory artifactStorePath = new FixedPath(System.getProperty("user.home") + "/.embeddedMongodbCustomPath");
	ITempNaming executableNaming = new UUIDTempNaming();

	Command command = Command.MongoD;

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaults(command)
		.artifactStore(new ArtifactStoreBuilder()
			.defaults(command)
			.download(new DownloadConfigBuilder()
				.defaultsForCommand(command)
				.artifactStorePath(artifactStorePath))
			.executableNaming(executableNaming))
		.build();

	MongodStarter runtime = MongodStarter.getInstance(runtimeConfig);
	MongodExecutable mongodExe = runtime.prepare(mongodConfig);
	...

### Usage - custom mongod process output

#### ... to console with line prefix

	...
	ProcessOutput processOutput = new ProcessOutput(Processors.namedConsole("[mongod>]"),
			Processors.namedConsole("[MONGOD>]"), Processors.namedConsole("[console>]"));

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaults(Command.MongoD)
		.processOutput(processOutput)
		.build();

	MongodStarter runtime = MongodStarter.getInstance(runtimeConfig);
	...

#### ... to file

	...
	IStreamProcessor mongodOutput = Processors.named("[mongod>]",
			new FileStreamProcessor(File.createTempFile("mongod", "log")));
	IStreamProcessor mongodError = new FileStreamProcessor(File.createTempFile("mongod-error", "log"));
	IStreamProcessor commandsOutput = Processors.namedConsole("[console>]");

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaults(Command.MongoD)
		.processOutput(new ProcessOutput(mongodOutput, mongodError, commandsOutput))
		.build();

	MongodStarter runtime = MongodStarter.getInstance(runtimeConfig);
	...

	...
	public class FileStreamProcessor implements IStreamProcessor {

		private FileOutputStream outputStream;

		public FileStreamProcessor(File file) throws FileNotFoundException {
			outputStream = new FileOutputStream(file);
		}

		@Override
		public void process(String block) {
			try {
				outputStream.write(block.getBytes());
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		@Override
		public void onProcessed() {
			try {
				outputStream.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

	}
	...

#### ... to java logging

	...
	Logger logger = Logger.getLogger(getClass().getName());

	ProcessOutput processOutput = new ProcessOutput(Processors.logTo(logger, Level.INFO), Processors.logTo(logger,
			Level.SEVERE), Processors.named("[console>]", Processors.logTo(logger, Level.FINE)));

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaultsWithLogger(Command.MongoD,logger)
		.processOutput(processOutput)
		.artifactStore(new ArtifactStoreBuilder()
			.defaults(Command.MongoD)
			.download(new DownloadConfigBuilder()
				.defaultsForCommand(Command.MongoD)
				.progressListener(new LoggingProgressListener(logger, Level.FINE))))
		.build();

	MongodStarter runtime = MongodStarter.getInstance(runtimeConfig);
	...

#### ... to default java logging (the easy way)

	...
	Logger logger = Logger.getLogger(getClass().getName());

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaultsWithLogger(Command.MongoD, logger)
		.build();

	MongodStarter runtime = MongodStarter.getInstance(runtimeConfig);
	...

### Custom Version

	...
	int port = 12345;
	IMongodConfig mongodConfig = new MongodConfigBuilder()
		.version(Versions.withFeatures(new GenericVersion("2.0.7-rc1"),Feature.SYNC_DELAY))
		.net(new Net(port, Network.localhostIsIPv6()))
		.build();

	MongodStarter runtime = MongodStarter.getDefaultInstance();
	MongodProcess mongod = null;

	MongodExecutable mongodExecutable = null;
	try {
		mongodExecutable = runtime.prepare(mongodConfig);
		mongod = mongodExecutable.start();

		...

	} finally {
		if (mongod != null) {
			mongod.stop();
		}
		if (mongodExecutable != null)
			mongodExecutable.stop();
	}
	...

### Main Versions

	IVersion version = Version.V2_2_5;
	// uses latest supported 2.2.x Version
	version = Version.Main.V2_2;
	// uses latest supported production version
	version = Version.Main.PRODUCTION;
	// uses latest supported development version
	version = Version.Main.DEVELOPMENT;

### Use Free Server Port

	Warning: maybe not as stable, as expected.

#### ... by hand

	...
	int port = Network.getFreeServerPort();
	...

#### ... automagic

	...
	IMongodConfig mongodConfig = new MongodConfigBuilder().version(Version.Main.PRODUCTION).build();

	MongodStarter runtime = MongodStarter.getDefaultInstance();

	MongodExecutable mongodExecutable = null;
	MongodProcess mongod = null;
	try {
		mongodExecutable = runtime.prepare(mongodConfig);
		mongod = mongodExecutable.start();

		MongoClient mongo = new MongoClient(new ServerAddress(mongodConfig.net().getServerAddress(), mongodConfig.net().getPort()));
		...

	} finally {
		if (mongod != null) {
			mongod.stop();
		}
		if (mongodExecutable != null)
			mongodExecutable.stop();
	}
	...

### ... custom timeouts

	...
	IMongodConfig mongodConfig = new MongodConfigBuilder()
		.version(Version.Main.PRODUCTION)
		.timeout(new Timeout(30000))
		.build();
	...

### Command Line Post Processing

	...
	ICommandLinePostProcessor postProcessor= ...

	IRuntimeConfig runtimeConfig = new RuntimeConfigBuilder()
		.defaults(Command.MongoD)
		.commandLinePostProcessor(postProcessor)
		.build();
	...

### Custom Command Line Options

	We changed the syncDelay to 0 which turns off sync to disc. To turn on default value used defaultSyncDelay().
	IMongodConfig mongodConfig = new MongodConfigBuilder()
	.version(Version.Main.PRODUCTION)
	.cmdOptions(new MongoCmdOptionsBuilder()
		.syncDeplay(10)
		.build())
	.build();
	...

### Snapshot database files from temp dir

	We changed the syncDelay to 0 which turns off sync to disc. To get the files to create an snapshot you must turn on default value (use defaultSyncDelay()).
	IMongodConfig mongodConfig = new MongodConfigBuilder()
	.version(Version.Main.PRODUCTION)
	.processListener(new ProcessListenerBuilder()
		.copyDbFilesBeforeStopInto(destination)
		.build())
	.cmdOptions(new MongoCmdOptionsBuilder()
		.defaultSyncDeplay()
		.build())
	.build();
	...

----

YourKit is kindly supporting open source projects with its full-featured Java Profiler.
YourKit, LLC is the creator of innovative and intelligent tools for profiling
Java and .NET applications. Take a look at YourKit's leading software products:
<a href="http://www.yourkit.com/java/profiler/index.jsp">YourKit Java Profiler</a> and
<a href="http://www.yourkit.com/.net/profiler/index.jsp">YourKit .NET Profiler</a>.
