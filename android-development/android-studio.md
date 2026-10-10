# **Android Studio**

## **Terms**

- **Activity** - An activity represents a single screen 
with a user interface just like window or frame of Java,
or a page in a web-app/web-site. Android activity is the
subclass of `ContextThemeWrapper` class.

- **Main Activity** - The "main" activity is the activity
that is launched first (the entry point declared with the
`MAIN`/`LAUNCHER` intent filter), and from it the rest of your application is opened.
Every application can have multiple activities, therefore
you can list other activities to load and use later on but
there is usually only one "main" activity.

- **Layout Editor** - Interactive Editor where we can create
our activity layout and design.
    - **Design** - This mode shows the default/current state
    of the **Layout** and **Views** components.
    - **Blueprint** - This shows every **Layout** and **Views** components (as outlines), even if some of them are in a hidden/invisible
    state.

- **Layout** - A box where we can put our **View** components
or another **Layout** (Nested layout). There are different types
of layout each having their own different way to arrange the
**View** components inside them.

- **Views** - The elements or component inside the **Layout**.
eg. Buttons, Text and etc.

- **Attributes** - Attributes or Properties are what describe a
**View** or a **Layout** (kind of like key-value pairs in **.JSON**). A **View** or
**Layout** has attributes inside them, in code both the **View**
and **Layout** can be described using the **.xml** markup language
(a language slightly similar to HTML).

- **Intent** - A class that provides functionality like transferring
data between activities.

<br><br>

## **About Project Structure**
- **.java** (or **.kt** for Kotlin) - files with this extension define the functionality.
- **.xml** - files with this extension define the **Layout, View**
and even acts like a **configuration** file like the **AndroidManifest.xml**.

<br><br>

## **ID Attribute**

Used to reference **Views** in **.java** code.

- **XML Syntax:**

    ```xml
    <ExampleViewTag
        android:id="@+id/ID_NAME"
    />
    ```

- **Accessing a specific ID in java code**

    ```java
    R.id.ID_NAME
    ```

- **Accessing other View elements**

    Note: In The **Layout Editor**, on the  **Component Tree** panel
    we can see the specific types of the **View** components that are
    used inside the selected activity layout. 

    ```java
    ViewClassType obj = findViewById(R.id.ID_NAME);
    ```

- **Getting a View object's own ID**

    ```java
    view.getId(); // called on a View instance
    ```

- **Getting user input from input field**

    ```java
    // input fields are `EditText` views (EditText is a subclass of TextView)
    EditText txtInput = findViewById(R.id.TXT_INPUT_ID);
    String input = txtInput.getText().toString();
    ```

<br><br>

## **Event Handling**

- **Creating an Event Handler method For a View**

    ```java
    public void EventHandlerName(View view) {
        view.getId();
        view.setEnabled(false);
        // ...
    }
    ```

- **Assigning Event Handler Method to Views**

    There are two ways to do this:

    1. Use the **Layout Editor**, click on the **View** where
    you want to use the event handler method, click on the
    **Attributes** panel, find the **onClick** event handler
    attribute, then select the event handler method
    (It should appear in the drop-down).

    2. Go to the designated **XML** file of your activity then
    find the **XML tag** of the **View** where you want to assign
    the event handler method then add a new attribute:

        ```xml
        <Button
            android:onClick="eventHandlerMethodName"
        />
        ```

    Always make sure you are editing the right XML file, remember
    that the **.xml** and **.java** files are linked with each other,
    If the event handler is not working or not appearing in the
    drop-down box of the **onClick** attribute, that might mean
    you are editing the wrong activity or XML file.

<br><br>

## **Enabling Auto Import**

**File > Settings > Editor > General > Auto Import > All/Always**

<br>

## **Logging**

Import the logging class

```java
import android.util.Log;
```

To log a message use the method below, the **Tag**
parameter will make your life easier when reading the logs,
by filtering the **Tag** string that you've used.

```java
Log.d("Tag", "Message");
```

<br><br>

## **Showing Alerts**

```java
import android.widget.Toast;
```

Alerts (called **Toasts**) are like pop up messages in the system itself,
they are the small ones that appear most of the time at the
bottom of the screen for a short time when something happens.

```java
Toast.makeText(this, "Alert Message", Toast.LENGTH_LONG).show();
```

<br><br>

## **Type Casting**

In android studio, an important use of casting to different types
is when we are casting the `View` class to its specific **view type**.

We do this in order to access the methods of those specific **view types** that
are not available in the base `View` class.

An example of a button event handler:

```java
package com.xxxxxx.yyyyyyyyyy;

import ...other imports...;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    
    // ...

    public void btnHandler(View btn_view) {
        Button btn = (Button) btn_view; // type casting
        Log.d("btnHandler Call", btn.getText().toString());
    }
}
```

Here if we access the method `.getText()` directly in the `btn_view`
it will not appear, so we needed to cast it first to the `Button`
class in order to access it.

<br><br>

## **String Resource**

Non-hard coded text that you can use across your application,
it might also provide some dynamism to the text display of your
application (maybe).

To add a string resource go to:

**Projects Panel > app > res > values > strings.xml**

then you can add a typical string resource item inside the
`<resources>` tag:

```xml
<string name="resource_name">the string</string>
```

**Using String Resource**

To use the string resource that you created, In the **Layout Editor**
go to the **Attributes** panel, choose an attribute where you want
the value to be the string resource that you created, then click the thin
vertical box beside it, then choose the string resource name you
want to use. (In XML it is referenced as `@string/resource_name`, and in Java as `getString(R.string.resource_name)`.)

<br><br>

## **Create new Activity**

On the project navigation panel, **right-click** the currently selected **app**/root project folder.

**app > right-click > New > Activity > Activity Boiler Plate**

This will automatically create a new **.java** and **.xml** file
for you, these files will represent the design and functionality
of that specific activity. This step will also link the two files
together and also edit the **AndroidManifest.xml** file automatically
to include the newly created **.xml** and **.java** files.

<br><br>

## **Changing The Main Activity**

**app > manifest > AndroidManifest.xml**

Move the `<intent-filter></intent-filter>` tag (the one containing the `android.intent.action.MAIN` action and `android.intent.category.LAUNCHER` category) inside the
`<activity></activity>` tag of the new activity that you want to
make as the new main activity.

You can also change the **icon image** and other things here.

<br><br>

## **Changing Activity Title**

**app > manifest > AndroidManifest.xml**
 
In the `<activity>` tag just add an `android:label="NEW_TITLE"` attribute.

```xml
<activity
android:label="NEW_TITLE">
<!-- tag content -->
</activity>
```

 **WARNING** : this
method will also **rename the actual app** if you applied it to
the **Main Activity**, to avoid this problem, you can use another
method where we will set the title of the main activity
programmatically, to do this just add a `setTitle("NEW_TITLE");`
inside the `public void onCreate()` method of the main activity
java class.

<br><br>

## **Launching Another Activity**

```java
import android.content.Intent;
```

```java
Intent intent = new Intent(this, JavaFileActivity.class);
startActivity(intent);
```

<br><br>

## **Persisting Data Across Activities**

- **Passing Data**

    ```java
    Intent i = new Intent(this, OtherActivity.class);
    
    // there are different overloads for the type of value.
    i.putExtra("KEY", VALUE);

    startActivity(i);
    ```

- **Grabbing Data** 

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ... boiler plate codes ...

        Intent i = getIntent();

        // here we assume that the data being passed is a string.
        // but remember that this is not always the case.
        String msg = i.getStringExtra("KEY");
    }
    ```


<br><br>

## **Activity Backwards Navigation**

Setting up a Parent-Child relationship between activities,
this will enable going back from a child activity to a 
parent activity.

To do this navigate to:

**app > manifest > AndroidManifest.xml**

Then select the `<activity>` tag that you want to be a child
then add this attribute:

```xml
<activity
    android:parentActivityName=".JavaFileActivity"
>
```

Here the **JavaFileActivity.java** will become the parent activity
of the child activity, a go back (up) arrow will appear in the child
activity prompting to return to the **JavaFileActivity**.

<br><br>

## **Remove an Activity in Android Studio**

**Select Activity .java > right-click > Find Usage > Select Usage > Delete Manually**

**Select Activity .xml > right-click > Find Usage > Select Usage > Delete Manually**

or

**Select Activity .java > right-click > Refactor > Safe Delete**

**Select Activity .xml > right-click > Refactor > Safe Delete**

<br><br>

## **Activity Life Cycle**

1. `onCreate()`

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    ```

2. `onStart()`

    ```java
    @Override
    protected void onStart() { // onStart() takes no parameters
        super.onStart();
    }
    ```

3. `onResume()`

    ```java
    @Override
    protected void onResume() {
        super.onResume();
    }
    ```

4. `onPause()`

    ```java
    @Override
    protected void onPause() {
        super.onPause();
    }
    ```

5. `onStop()`

    ```java
    @Override
    protected void onStop() {
        super.onStop();
    }
    ```

6.  `onDestroy()`

    ```java
    @Override
    protected void onDestroy() {
        super.onDestroy();
    }
    ```

7. `onRestart()`

    ```java
    @Override
    protected void onRestart() {
        super.onRestart();
    }
    ```

[![Activity Life Cycle](./activity_lifecycle.png)](https://developer.android.com/guide/components/activities/activity-lifecycle#alc)

**Different Calls**

- When the app is **STARTED**

    `> onCreate() > onStart() > onResume()`

- When we **LEAVE** the app (HOME or RECENT BUTTON)

    `> onPause() > onStop()`

- When we **GO BACK** to it after leaving

    `> onRestart() > onStart() > onResume()`

- When we **EXIT** the app (BACK, REMOVE IN RECENT or APP EXIT)

    `> onPause() > onStop() > onDestroy()`

    (NOTE: since Android 12, pressing BACK on the app's root launcher activity no longer destroys it, it just moves the app to the background: `> onPause() > onStop()`)

- When we **OPEN ANOTHER ACTIVITY** that **completely hides** the old activity

    `> onPause() > onStop()`

- When we **GO BACK** to the old activity that was **completely hidden**.

    `> onRestart() > onStart() > onResume()`

- When we **OPEN ANOTHER ACTIVITY** but the old activity is still partially visible (new activity has a **transparent layout** or is dialog-themed)

    `> onPause()`

- When we **GO BACK** to the **old activity** from the new activity that has a **transparent layout**

    `> onResume()`
