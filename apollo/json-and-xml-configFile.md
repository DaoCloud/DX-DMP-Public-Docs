# json/xml configuration file manipulation documentation

If apollo-client is introduced, add the following code to get the xml and json files where needed

1.Get the xml type configuration file

```
//get the datasources.xml file
xmlConfigFile = ConfigService.getConfigFile("datasources",ConfigFileFormat.XML);
// Add a listener
xmlConfigFile.addChangeListener(new ConfigFileChangeListener() {
  @Override
  public void onChange(ConfigFileChangeEvent changeEvent) {
    logger.info(changeEvent.toString());
  }
});
xmlConfigFile.getContent();  // Get the configuration content of the xml
... Business Processing
```

2.Get the configuration file of json type
```
//Get the datasources.json file
jsonConfigFile = ConfigService.getConfigFile("datasources",ConfigFileFormat.JSON);
// Add a listener
jsonConfigFile.addChangeListener(new ConfigFileChangeListener() {
  @Override
  public void onChange(ConfigFileChangeEvent changeEvent) {
    logger.info(changeEvent.toString());
  }
});
jsonConfigFile.getContent(); // Get the configuration content of json
.... Business Processing
```
