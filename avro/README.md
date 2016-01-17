# AVRO Schema evolution
[Avro Schema Resolution](https://avro.apache.org/docs/1.7.7/spec.html#Schema+Resolution)

Avro exercise to demonstrate how schema evolution works through a simple example. 

### Create data and schema files

Create *quotes.json* file:
```
{"a":"Superman (Clark Kent)","b":"I'm here to fight for truth, and justice, and the American way."}
{"a":"Kick-Ass (Dave Lizewski)","b":"Like every serial killer already knew: eventually fantasising just doesn't do it for you anymore."}
{"a":"Captain America (Steve Rogers)","b":"I punched out Adolf Hitler 200 times."}
{"a":"Batman (Bruce Wayne)","b":"You wanna get nuts? Come on! Let's get nuts!"}
{"a":"Wolverine (Logan)","b":"You know, sometimes when you cage the beast, the beast gets angry."}
{"a":"Daredevil (Matt Murdock)","b":"I hope justice is found here today... before justice finds you."}
{"a":"Iron Man (Tony Stark)","b":"Give me a scotch. I'm starving."}
{"a":"Thor","b":"Our ancestors called it magic but you call it science. I come from a land where they are one and the same."}
{"a":"Spider-Man (Peter Parker)","b":"This is my gift, my curse. Who am I? I'm Spider-man."}
{"a":"Catwoman (Patience Philips)","b":"Cats come when they feel like it. Not when they're told."}
{"a":"Mr. Incredible (Bob Parr)","b":"No matter how many times you save the world, it always manages to get back in jeopardy again."}
{"a":"Hancock","b":"I'll break my foot off in your ass, woman..."}
{"a":"Superman (Clark Kent)","b":"I hear everything. You wrote that the world doesn't need a savior, but every day I hear people crying for one."}
{"a":"Mr. Fantastic (Reed Richards)","b":"I stayed in and studied like a good little nerd. And fifteen years later, I'm one of the greatest minds of the 21st century."}
{"a":"Hulk (Bruce Banner)","b":"Hulk, SMASH!"}
{"a":"Batman (Bruce Wayne)","b":"Sometimes the truth isn't good enough, sometimes people deserve more. Sometimes people deserve to have their faith rewarded..."}
{"a":"Green Lantern (Hal Jordan)","b":"I know what it's like. To not live up to expectations. To feel like nothing that you do will ever be good enough."}
{"a":"Ghost Rider (Johnny Blaze)","b":"He may have my soul but he doesn't have my spirit."}
{"a":"Robin (Dick Grayson)","b":"I want a car, chicks dig the car."}
{"a":"Blade","b":"Some motherf***ers are always trying to ice-skate uphill."}
{"a":"Hellboy","b":"Didn't I kill you already?"}
{"a":"Professor X (Charles Xavier)","b":"Wise man say forgiveness is divine, but never pay full price for late pizza."}
```

Create *quotes.avsc* file:
```
{
  "type": "record",
  "name": "quotes",
  "namespace": "com.superhero",
  "fields": [{
    "name": "a",
    "type": "string",
    "doc": "Name"
  }, {
    "name": "b",
    "type": "string",
    "doc": "quote"
  }],
  "doc:": "SuperHero quotes"
}
```

### Create Avro data file from the JSON and the Schema
`avro-tools fromjson quotes.json --schema-file quotes.avsc > quotes.avro`

Validate the AVRO data file:
`avro-tools tojson quotes.avro --pretty`
`avro-tools getschema quotes.avro`
`avro-tools getmeta quotes.avro`

## Alter the original schema
We decided that we need to track when did the SuperHero said what's said...
We have two options to add the new column - create add it as optional or add it with a default value:

Add an optinal column to the schema (*quotes_v2_op1.avsc*):
```
{
  "type": "record",
  "name": "quotes",
  "namespace": "com.superhero",
  "fields": [{
    "name": "a",
    "type": "string",
    "doc": "Name"
  }, {
    "name": "b",
    "type": "string",
    "doc": "quote"
  }, {
    "name": "c",
    "type": ["null", "long"],
    "default": null,
    "doc": "time"
  }],
  "doc:": "SuperHero quotes"
}
```

Add an default value to the schema (*quotes_v2_op2.avsc*):
```
{
  "type": "record",
  "name": "quotes",
  "namespace": "com.superhero",
  "fields": [{
    "name": "a",
    "type": "string",
    "doc": "Name"
  }, {
    "name": "b",
    "type": "string",
    "doc": "quote"
  }, {
    "name": "c",
    "type": "long",
    "default": 1,
    "doc": "time"
  }],
  "doc:": "SuperHero quotes"
}
```

## Now let's test the schemas with Hive

First move everything to HDFS
`sudo -u hdfs hdfs dfs -mkdir -p /data/avro_test/schema`
`sudo -u hdfs hdfs dfs -mkdir /data/avro_test/data/`
`sudo -u hdfs hdfs dfs -put /root/quotes.avsc /root/quotes_v2_op1.avsc /root/quotes_v2_op2.avsc /data/avro_test/schema`
`sudo -u hdfs hdfs dfs -put /root/quotes.avro /data/avro_test/data/`

Connect to Hive from shell
`beeline -u jdbc:hive2://localhost:10000`

Create the tables to read with the altered schemas from the Avro dataset:
```
CREATE TABLE quotes
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED as INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
TBLPROPERTIES ('avro.schema.url'='hdfs:///data/avro_test/schema/quotes_v2_op1.avsc');

LOAD DATA
INPATH '/data/avro_test/data/'
INTO TABLE quotes;
```



