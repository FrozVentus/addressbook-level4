# darrinloh
###### /java/seedu/address/storage/StorageManager.java
``` java
    public StorageManager(AddressBookStorage addressBookStorage, UserPrefsStorage userPrefsStorage) {
        super();
        this.addressBookStorage = addressBookStorage;
        this.userPrefsStorage = userPrefsStorage;

        Optional<AddressBookData> addressBookOptional;
        try {
            addressBookOptional = addressBookStorage.readAddressBook();
            if (addressBookOptional.isPresent()) {
                backUpAddressBook(addressBookOptional.get());
                logger.info("AddressBook found, backup initiated.");
            } else {
                logger.warning("AddressBook not found, backup will not initiate.");
            }
        } catch (DataConversionException e) {
            logger.warning("Data file not in the correct format. Backup will not initiate.");
        } catch (IOException e) {
            logger.warning("Problem while reading from the file. Backup will not initiate.");
        }

    }
```
###### /java/seedu/address/storage/StorageManager.java
``` java

    // ================ UserPrefs methods ==============================

    @Override
    public String getUserPrefsFilePath() {
        return userPrefsStorage.getUserPrefsFilePath();
    }

    @Override
    public Optional<UserPrefs> readUserPrefs() throws DataConversionException, IOException {
        return userPrefsStorage.readUserPrefs();
    }

    @Override
    public void saveUserPrefs(UserPrefs userPrefs) throws IOException {
        userPrefsStorage.saveUserPrefs(userPrefs);
    }


    // ================ AddressBook methods ==============================

    @Override
    public String getAddressBookFilePath() {
        return addressBookStorage.getAddressBookFilePath();
    }

```
###### /java/seedu/address/storage/StorageManager.java
``` java
    @Override
    public String getBackUpAddressBookFilePath() {
        return addressBookStorage.getAddressBookFilePath() + "-backup.xml";
    }


    @Override
    public Optional<AddressBookData> readBackUpAddressBook() throws DataConversionException, IOException {
        return readAddressBook(addressBookStorage.getAddressBookFilePath() + "-backup.xml");
    }
```
###### /java/seedu/address/storage/StorageManager.java
``` java

    @Override
    public Optional<AddressBookData> readAddressBook() throws DataConversionException, IOException {
        return readAddressBook(addressBookStorage.getAddressBookFilePath());
    }

    @Override
    public Optional<AddressBookData> readAddressBook(String filePath) throws DataConversionException, IOException {
        logger.fine("Attempting to read data from file: " + filePath);
        return addressBookStorage.readAddressBook(filePath);
    }

    @Override
    public void saveAddressBook(AddressBookData addressBook) throws IOException {
        saveAddressBook(addressBook, addressBookStorage.getAddressBookFilePath());
    }

    @Override
    public void saveAddressBook(AddressBookData addressBook, String filePath)
            throws IOException {
        logger.fine("Attempting to write to data file: " + filePath);
        addressBookStorage.saveAddressBook(addressBook, filePath);
    }

```
###### /java/seedu/address/storage/StorageManager.java
``` java
    private void backUpAddressBook(AddressBookData addressBook) throws IOException {
        saveAddressBook(addressBook, getLocalBackUpAddressBookFilePath());
    }

    public String getLocalBackUpAddressBookFilePath() {
        return addressBookStorage.getAddressBookFilePath() + "-backup.xml";
    }
```
###### /java/seedu/address/storage/StorageManager.java
``` java

    @Override
    @Subscribe
    public void handleAddressBookChangedEvent(AddressBookChangedEvent event) {
        logger.info(LogsCenter.getEventHandlingLogMessage(event, "Local data changed, saving to file"));
        try {
            saveAddressBook(event.data);
        } catch (IOException e) {
            raise(new DataSavingExceptionEvent(e));
        }
    }

}
```
###### /java/seedu/address/MainApp.java
``` java
    private Model initModelManager(Storage storage, UserPrefs userPrefs) {
        Optional<AddressBookData> addressBookOptional;
        AddressBookData initialData;
        try {
            addressBookOptional = storage.readAddressBook();
            if (!addressBookOptional.isPresent()) {
                logger.info("Data file not found. Will be starting with a sample AddressBook");
                ReadOnlyAddressBook addressBook = SampleDataUtil.getSampleAddressBook();
                AddressBookData data = new AddressBookData(addressBook, new AddressBook());
                initialData = data;
            } else {
                initialData = addressBookOptional.get();
            }
        } catch (DataConversionException e) {
            logger.warning("Data file not in the correct format. Will start backup file");
            try {
                addressBookOptional = storage.readBackUpAddressBook();
                initialData = addressBookOptional.get();
            } catch (DataConversionException f) {
                logger.warning("Backup file is not in the correct format."
                        + "Will be starting with an empty AddressBook");
                initialData = new AddressBookData();
            } catch (IOException f) {
                logger.warning("Problem while reading from the backup file."
                        + "Will be starting with an empty AddressBook");
                initialData = new AddressBookData();
            }
        } catch (IOException e) {
            logger.warning("Problem while reading from the file. Will be starting with an empty AddressBook");
            initialData = new AddressBookData();
        }

        return new ModelManager(initialData.getAddressBook(), initialData.getRecycleBin(), userPrefs);
    }
```
###### /java/seedu/address/MainApp.java
``` java


    private void initLogging(Config config) {
        LogsCenter.init(config);
    }

    /**
     * Returns a {@code Config} using the file at {@code configFilePath}. <br>
     * The default file path {@code Config#DEFAULT_CONFIG_FILE} will be used instead
     * if {@code configFilePath} is null.
     */
    protected Config initConfig(String configFilePath) {
        Config initializedConfig;
        String configFilePathUsed;

        configFilePathUsed = Config.DEFAULT_CONFIG_FILE;

        if (configFilePath != null) {
            logger.info("Custom Config file specified " + configFilePath);
            configFilePathUsed = configFilePath;
        }

        logger.info("Using config file : " + configFilePathUsed);

        try {
            Optional<Config> configOptional = ConfigUtil.readConfig(configFilePathUsed);
            initializedConfig = configOptional.orElse(new Config());
        } catch (DataConversionException e) {
            logger.warning("Config file at " + configFilePathUsed + " is not in the correct format. "
                    + "Using default config properties");
            initializedConfig = new Config();
        }

        //Update config file in case it was missing to begin with or there are new/unused fields
        try {
            ConfigUtil.saveConfig(initializedConfig, configFilePathUsed);
        } catch (IOException e) {
            logger.warning("Failed to save config file : " + StringUtil.getDetails(e));
        }
        return initializedConfig;
    }

    /**
     * Returns a {@code UserPrefs} using the file at {@code storage}'s user prefs file path,
     * or a new {@code UserPrefs} with default configuration if errors occur when
     * reading from the file.
     */
    protected UserPrefs initPrefs(UserPrefsStorage storage) {
        String prefsFilePath = storage.getUserPrefsFilePath();
        logger.info("Using prefs file : " + prefsFilePath);

        UserPrefs initializedPrefs;
        try {
            Optional<UserPrefs> prefsOptional = storage.readUserPrefs();
            initializedPrefs = prefsOptional.orElse(new UserPrefs());
        } catch (DataConversionException e) {
            logger.warning("UserPrefs file at " + prefsFilePath + " is not in the correct format. "
                    + "Using default user prefs");
            initializedPrefs = new UserPrefs();
        } catch (IOException e) {
            logger.warning("Problem while reading from the file. Will be starting with an empty AddressBook");
            initializedPrefs = new UserPrefs();
        }

        //Update prefs file in case it was missing to begin with or there are new/unused fields
        try {
            storage.saveUserPrefs(initializedPrefs);
        } catch (IOException e) {
            logger.warning("Failed to save config file : " + StringUtil.getDetails(e));
        }

        return initializedPrefs;
    }

    private void initEventsCenter() {
        EventsCenter.getInstance().registerHandler(this);
    }

    @Override
    public void start(Stage primaryStage) {
        logger.info("Starting OneBook " + MainApp.VERSION);
        ui.start(primaryStage);
    }

    @Override
    public void stop() {
        logger.info("============================ [ Stopping Address Book ] =============================");
        ui.stop();
        try {
            storage.saveUserPrefs(userPrefs);
        } catch (IOException e) {
            logger.severe("Failed to save preferences " + StringUtil.getDetails(e));
        }
        Platform.exit();
        System.exit(0);
    }

    @Subscribe
    public void handleExitAppRequestEvent(ExitAppRequestEvent event) {
        logger.info(LogsCenter.getEventHandlingLogMessage(event));
        this.stop();
    }

    public static void main(String[] args) {
        launch(args);
    }
}
```
