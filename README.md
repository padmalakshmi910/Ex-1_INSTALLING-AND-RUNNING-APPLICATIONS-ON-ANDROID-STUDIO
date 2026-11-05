# Ex-1_INSTALLING-AND-RUNNING-APPLICATIONS-ON-ANDROID-STUDIO

# AIM

To develop, install, and run an Android application named Habit Bloom using Android Studio to manage daily habits and track their completion progress.

# PROCEDURE

1. Open Android Studio and create a new Android project.
2. Design the application interface with a suitable layout.
3. Add TextViews to display the application title and daily progress.
4. Add CheckBoxes to display daily habits such as reading, drinking water, and studying.
5. Add a Button to create new habits.
6. Implement functionality to mark habits as completed and delete unwanted habits.
7. Calculate and display the daily progress based on completed habits.
8. Build and run the application using an Android emulator or physical device.
9. Verify that the application works correctly.

# PROGRAM

## AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.HabitBloom">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## MainActivity.java

```
package com.example.habitbloom;

import android.app.AlertDialog;
import android.graphics.Color;
import android.graphics.Typeface;
import android.graphics.drawable.GradientDrawable;
import android.os.Bundle;
import android.view.Gravity;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.ProgressBar;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import org.json.JSONArray;
import org.json.JSONObject;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    LinearLayout habitContainer;
    ProgressBar progressBar;
    TextView progressText;
    Button addHabitButton;

    ArrayList<String> habitNames = new ArrayList<>();
    ArrayList<Boolean> habitDone = new ArrayList<>();

    android.content.SharedPreferences preferences;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        habitContainer = findViewById(R.id.habitContainer);
        progressBar = findViewById(R.id.progressBar);
        progressText = findViewById(R.id.progressText);
        addHabitButton = findViewById(R.id.addHabitButton);

        preferences = getSharedPreferences(
                "HabitBloomData", MODE_PRIVATE);

        loadHabits();

        if (habitNames.isEmpty()) {
            addHabit("Read for 20 minutes");
            addHabit("Drink enough water");
            addHabit("Study for one hour");
        }

        displayHabits();

        addHabitButton.setOnClickListener(
                v -> showAddHabitDialog());
    }

    void addHabit(String name) {
        habitNames.add(name);
        habitDone.add(false);
        saveHabits();
    }

    void showAddHabitDialog() {
        EditText input = new EditText(this);
        input.setHint("Enter your new habit");

        new AlertDialog.Builder(this)
                .setTitle("Create a Habit")
                .setView(input)
                .setPositiveButton("Add", (dialog, which) -> {
                    String name = input.getText()
                            .toString().trim();

                    if (!name.isEmpty()) {
                        addHabit(name);
                        displayHabits();

                        Toast.makeText(this,
                                "Habit added!",
                                Toast.LENGTH_SHORT).show();
                    }
                })
                .setNegativeButton("Cancel", null)
                .show();
    }

    void displayHabits() {
        habitContainer.removeAllViews();

        for (int i = 0; i < habitNames.size(); i++) {
            final int position = i;

            LinearLayout row = new LinearLayout(this);
            row.setOrientation(LinearLayout.HORIZONTAL);
            row.setGravity(Gravity.CENTER_VERTICAL);
            row.setPadding(12, 8, 8, 8);

            GradientDrawable background =
                    new GradientDrawable();

            background.setColor(Color.WHITE);
            background.setCornerRadius(24);
            background.setStroke(2,
                    Color.rgb(224, 234, 220));

            row.setBackground(background);

            LinearLayout.LayoutParams rowParams =
                    new LinearLayout.LayoutParams(
                            LinearLayout.LayoutParams.MATCH_PARENT,
                            LinearLayout.LayoutParams.WRAP_CONTENT);

            rowParams.setMargins(0, 0, 0, 14);
            row.setLayoutParams(rowParams);

            CheckBox checkBox = new CheckBox(this);
            checkBox.setText(habitNames.get(i));
            checkBox.setTextColor(Color.rgb(36, 76, 53));
            checkBox.setTextSize(15);
            checkBox.setTypeface(null, Typeface.BOLD);

            checkBox.setLayoutParams(
                    new LinearLayout.LayoutParams(
                            0,
                            LinearLayout.LayoutParams.WRAP_CONTENT,
                            1));

            checkBox.setChecked(habitDone.get(i));

            checkBox.setOnCheckedChangeListener(
                    (button, checked) -> {
                        habitDone.set(position, checked);
                        saveHabits();
                        updateProgress();
                    });

            Button deleteButton = new Button(this);
            deleteButton.setText("X");
            deleteButton.setTextSize(12);
            deleteButton.setTextColor(
                    Color.rgb(150, 70, 70));

            row.addView(checkBox);
            row.addView(deleteButton);
            habitContainer.addView(row);

            deleteButton.setOnClickListener(v -> {
                habitNames.remove(position);
                habitDone.remove(position);
                saveHabits();
                displayHabits();
            });
        }

        updateProgress();
    }

    void updateProgress() {
        int completed = 0;

        for (boolean done : habitDone) {
            if (done) {
                completed++;
            }
        }

        int total = habitNames.size();

        int percent = total == 0
                ? 0 : completed * 100 / total;

        progressBar.setProgress(percent);

        progressText.setText(
                completed + " of " + total
                        + " habits completed");
    }

    void saveHabits() {
        JSONArray array = new JSONArray();

        try {
            for (int i = 0; i < habitNames.size(); i++) {
                JSONObject item = new JSONObject();

                item.put("name", habitNames.get(i));
                item.put("done", habitDone.get(i));

                array.put(item);
            }

            preferences.edit()
                    .putString("habits", array.toString())
                    .apply();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    void loadHabits() {
        String saved = preferences.getString(
                "habits", "");

        if (saved.isEmpty()) {
            return;
        }

        try {
            JSONArray array = new JSONArray(saved);

            for (int i = 0; i < array.length(); i++) {
                JSONObject item = array.getJSONObject(i);

                habitNames.add(item.getString("name"));
                habitDone.add(item.getBoolean("done"));
            }

        } catch (Exception e) {
            habitNames.clear();
            habitDone.clear();
        }
    }
}

```

## activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#F5F7F0"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="22dp">

        <!-- Header -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Habit Bloom 🌿"
            android:textColor="#24543A"
            android:textSize="30sp"
            android:textStyle="bold" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="6dp"
            android:text="Grow better, one day at a time."
            android:textColor="#778477"
            android:textSize="15sp" />

        <!-- Date Heading -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="30dp"
            android:text="YOUR DAILY JOURNEY"
            android:textColor="#66816B"
            android:textSize="12sp"
            android:textStyle="bold"
            android:letterSpacing="0.12" />

        <!-- Progress Card -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="12dp"
            android:background="#E3EDE0"
            android:orientation="vertical"
            android:padding="20dp">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Today's progress 🌱"
                android:textColor="#24543A"
                android:textSize="18sp"
                android:textStyle="bold" />

            <TextView
                android:id="@+id/progressText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                android:text="0 of 3 habits completed"
                android:textColor="#486A50"
                android:textSize="15sp" />

            <ProgressBar
                android:id="@+id/progressBar"
                style="?android:attr/progressBarStyleHorizontal"
                android:layout_width="match_parent"
                android:layout_height="10dp"
                android:layout_marginTop="14dp"
                android:max="100"
                android:progress="0"
                android:progressTint="#43885B"
                android:progressBackgroundTint="#C8D9C4" />

        </LinearLayout>

        <!-- Habits Heading -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="30dp"
            android:text="Today's habits"
            android:textColor="#24543A"
            android:textSize="24sp"
            android:textStyle="bold" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:text="Small actions make a big difference."
            android:textColor="#778477"
            android:textSize="14sp" />

        <!-- Habit List -->
        <LinearLayout
            android:id="@+id/habitContainer"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="18dp"
            android:orientation="vertical" />

        <!-- Add Habit Button -->
        <Button
            android:id="@+id/addHabitButton"
            android:layout_width="match_parent"
            android:layout_height="58dp"
            android:layout_marginTop="12dp"
            android:backgroundTint="#326B46"
            android:text="+   Add New Habit"
            android:textAllCaps="false"
            android:textColor="#FFFFFF"
            android:textSize="16sp"
            android:textStyle="bold" />

        <!-- Footer -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="28dp"
            android:gravity="center"
            android:paddingBottom="16dp"
            android:text="Be patient. Keep growing. 🌱"
            android:textColor="#778477"
            android:textSize="14sp" />

    </LinearLayout>

</ScrollView>

```

# OUTPUT

The Habit Bloom application was successfully installed and executed in Android Studio. It displays daily habits with checkboxes, allows users to add and delete habits, and shows the completion progress.

<img width="357" height="737" alt="Screenshot 2026-11-05 103312" src="https://github.com/user-attachments/assets/a72f28b3-e656-48b0-b0e5-151fe8638898" />


# RESULT

Thus, the Habit Bloom Android application was successfully developed, installed, and executed using Android Studio.
