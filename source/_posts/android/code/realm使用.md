---
title:  Realm使用笔记
date: 

categories: 
- android 
- code
tags:  
- android
- code
---


1.

```
// Create a RealmConfiguration that saves the Realm file in the app's "files" directory.
RealmConfiguration realmConfig = new RealmConfiguration.Builder(context).build();
Realm.setDefaultConfiguration(realmConfig);
// Get a Realm instance for this thread
Realm realm = Realm.getDefaultInstance();
```
```
// The RealmConfiguration is created using the builder pattern.
// The Realm file will be located in Context.getFilesDir() with name "myrealm.realm"
RealmConfiguration config = new RealmConfiguration.Builder(context)
  .name("myrealm.realm")
  .encryptionKey(getKey())
  .schemaVersion(42)
  .modules(new MySchemaModule())
  .migration(new MyMigration())
  .build();
// Use the config
Realm realm = Realm.getInstance(config);
```
```
RealmConfiguration myConfig = new RealmConfiguration.Builder(context)
  .name("myrealm.realm")
  .schemaVersion(2)
  .modules(new MyCustomSchema())
  .build();
RealmConfiguration otherConfig = new RealmConfiguration.Builder(context)
  .name("otherrealm.realm")
  .schemaVersion(5)
  .modules(new MyOtherSchema())
  .build();
Realm myRealm = Realm.getInstance(myConfig);
Realm otherRealm = Realm.getInstance(otherConfig);
public class MyApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    // The Realm file will be located in Context.getFilesDir() with name "default.realm"
    RealmConfiguration config = new RealmConfiguration.Builder(this).build();
    Realm.setDefaultConfiguration(config);
  }
}
```
```
public class MyActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Realm realm = Realm.getDefaultInstance();
    // ... Do something ...
    realm.close();
  }
}
​````

```
@RealmClass
public class User implements RealmModel {
}
// With RealmObject
user.isValid();
user.addChangeListener(listener);
// With RealmModel
RealmObject.isValid(user);
RealmObject.addChangeListener(user, listener);
```

```
final MyObject obj = new MyObject();
obj.setId(42);
obj.setName("Fish");
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        // This will create a new object in Realm or throw an exception if the
        // object already exists (same primary key)
        // realm.copyToRealm(obj);
        // This will update an existing object with the same primary key
        // or create a new object if an object with no primary key = 42
        realm.copyToRealmOrUpdate(obj);
    }
});
```

```
realm.beginTransaction();
User user = realm.createObject(User.class); // Create a new object
user.setName("John");
user.setEmail("john@corporation.com");
realm.commitTransaction();
```

```
User user = new User("John");
user.setEmail("john@corporation.com");
// Copy the object to Realm. Any further changes must happen on realmUser
realm.beginTransaction();
User realmUser = realm.copyToRealm(user);
realm.commitTransaction();
```

```
// Query Realm for all dogs younger than 2 years old
final RealmResults<Dog> puppies = realm.where(Dog.class).lessThan("age", 2).findAll();
puppies.size(); // => 0 because no dogs have been added to the Realm yet
// Persist your data in a transaction
realm.beginTransaction();
final Dog managedDog = realm.copyToRealm(dog); // Persist unmanaged objects
Person person = realm.createObject(Person.class); // Create managed objects directly
person.getDogs().add(managedDog);
realm.commitTransaction();
public void onStop () {
    if (transaction != null && !transaction.isCancelled()) {
        transaction.cancel();
    }
}
```

​````
// Listeners will be notified when data changes
puppies.addChangeListener(new RealmChangeListener<RealmResults<Dog>>() {
    @Override
    public void onChange(RealmResults<Dog> results) {
        // Query results are updated in real time
        puppies.size(); // => 1
    }
});

// Asynchronously update objects on a background thread
realm.executeTransactionAsync(new Realm.Transaction() {
    @Override
    public void execute(Realm bgRealm) {
        Dog dog = bgRealm.where(Dog.class).equalTo("age", 1).findFirst();
        dog.setAge(3);
    }
}, new Realm.Transaction.OnSuccess() {
    @Override
    public void onSuccess() {
    	// Original queries and Realm objects are automatically updated.
    	puppies.size(); // => 0 because there are no more puppies younger than 2 years old
    	managedDog.getAge();   // => 3 the dogs age is updated
    }
});

realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        Dog myDog = realm.createObject(Dog.class);
        myDog.setName("Fido");
        myDog.setAge(1);
    }
});
​````
```
Dog myDog = realm.where(Dog.class).equalTo("age", 1).findFirst();
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        Dog myPuppy = realm.where(Dog.class).equalTo("age", 1).findFirst();
        myPuppy.setAge(2);
    }
});
myDog.getAge(); // => 2
```

```
realm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm bgRealm) {
                User user = bgRealm.createObject(User.class);
                user.setName("John");
                user.setEmail("john@corporation.com");
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                // Transaction was a success.
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                // Transaction failed and was automatically canceled.
            }
        });
public void onStop () {
    if (transaction != null && !transaction.isCancelled()) {
        transaction.cancel();
    }
}
```

```
// r1 => [U1,U2]
RealmResults<Person> r1 = realm.where(Person.class)
                             .equalTo("dogs.name", "Fluffy")
                             .equalTo("dogs.color", "Brown")
                             .findAll();
// r2 => [U2]
RealmResults<Person> r2 = realm.where(Person.class)
                             .equalTo("dogs.name", "Fluffy")
                             .findAll()
                             .where()
                             .equalTo("dogs.color", "Brown")
                             .findAll();
                             .where()
                             .equalTo("dogs.color", "Yellow")
                             .findAll();
// Build the query looking at all users:
RealmQuery<User> query = realm.where(User.class);
// Add query conditions:
query.equalTo("name", "John");
query.or().equalTo("name", "Peter");
// Execute the query:
RealmResults<User> result1 = query.findAll();
// Or alternatively do the same all at once (the "Fluent interface"):
RealmResults<User> result2 = realm.where(User.class)
                                  .equalTo("name", "John")
                                  .or()
                                  .equalTo("name", "Peter")
                                  .findAll();
RealmResults<User> r = realm.where(User.class)
                            .greaterThan("age", 10)  //implicit AND
                            .beginGroup()
                                .equalTo("name", "Peter")
                                .or()
                                .contains("name", "Jo")
                            .endGroup()
                            .findAll();
// r1 => [U1,U2]
RealmResults<Person> r1 = realm.where(Person.class)
                             .equalTo("dogs.name", "Fluffy")
                             .equalTo("dogs.color", "Brown")
                             .findAll();
// r2 => [U2]
RealmResults<Person> r2 = realm.where(Person.class)
                             .equalTo("dogs.name", "Fluffy")
                             .findAll()
                             .where()
                             .equalTo("dogs.color", "Brown")
                             .findAll();
                             .where()
                             .equalTo("dogs.color", "Yellow")
                             .findAll();
6
final RealmResults<User> users = getUsers();
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        users.deleteFromRealm(0); // Delete and remove object directly
    }
});
for (User user : users) {
    showUser(user); // Deleted user will not be shown
}
// obtain the results of a query
final RealmResults<Dog> results = realm.where(Dog.class).findAll();
// All changes to data must happen in a transaction
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        // remove single match
        results.deleteFirstFromRealm();
        results.deleteLastFromRealm();
        // remove a single object
        Dog dog = results.get(5);
        dog.deleteFromRealm();
        // Delete all matches
        results.deleteAllFromRealm();
    }
});
As
7.
/ Setup Realm in your Application
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        RealmConfiguration realmConfiguration = new RealmConfiguration.Builder(this).build();
        Realm.setDefaultConfiguration(realmConfiguration);
    }
}
// onCreate()/onDestroy() overlap when switching between activities so onCreate()
// on Activity 2 will be called before onDestroy() on Activity 1.
public class MyActivity extends Activity {
    private Realm realm;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        realm = Realm.getDefaultInstance();
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        realm.close();
    }
}
// Use onStart()/onStop() for Fragments as onDestroy() might not be called.
public class MyFragment extends Fragment {
    private Realm realm;
    @Override
    public void onStart() {
        super.onStart();
        realm = Realm.getDefaultInstance();
    }
    @Override
    public void onStop() {
        super.onStop();
        realm.close();
    }
}

.
````

````