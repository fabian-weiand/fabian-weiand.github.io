[Home](index.md) - [Code Review](code_review.md) - [Data Structures and Algorithms](algorithms_data_structures.md) - [Software Engineering and Design](software_design_engineering.md) - [Databases](databases.md)

## Databases

### About

This artifact is an android app for weight loss written in java using the android studio development environment during the Mobile Architecture and Programming class in the program. 
It showcases my skills In using Android studio to design and implement the layers of an mobile application. This program includes a front end for the user to interact with, the backend database to store user data, and the driver in between.

### The code

```
package com.zybooks.weighttrackerbybiz;

//HEADER INCLUSIONS
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import android.Manifest;
import android.content.Intent;
import android.content.SharedPreferences;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.os.Bundle;
import android.telephony.SmsManager;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    //SET LOCAL VARIABLES
    private Button eTargetButton;
    private Button cWeightButton;
    private TextView tHistory;
    private TextView tSignOut;
    private TextView cWeight;
    private TextView tWeight;
    private String cWeightText;
    private String tWeightText;
    private String sCellNumber;
    DataBaseWeight Weightdb;
    private StringBuilder weightHistory;
    private Cursor data;
    private TextView mScrollText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //REQUEST TEXT MESSAGE PERMISSIONS
        ActivityCompat.requestPermissions(MainActivity.this, new String[]
                {Manifest.permission.SEND_SMS,
                        Manifest.permission.READ_SMS},
                PackageManager.PERMISSION_GRANTED);

        //TIE ACTIVITY ID'S TO LOCAL VARIABLES
        eTargetButton = (Button) findViewById(R.id.buttonTarget);
        cWeightButton = (Button) findViewById(R.id.buttonCurrent);
        tHistory = (TextView) findViewById(R.id.textHistory);
        cWeight = (TextView) findViewById(R.id.weightDis1);
        tWeight = (TextView) findViewById(R.id.weightDis2);
        mScrollText = (TextView) findViewById(R.id.mainScrollText);
        tSignOut = (TextView) findViewById(R.id.signOut);

        //SET CURRENT WEIGHT DISPLAY
        SharedPreferences setCWeight = getSharedPreferences("myPref",0);
        if(setCWeight.contains("Key_1")){
            cWeightText = setCWeight.getString("Key_1",null);
        } else {
            cWeightText = null;
        }
        cWeight.setText(cWeightText);

        //SET TARGET WEIGHT DISPLAY
        SharedPreferences settWeight = getSharedPreferences("myPref",0);
        if(settWeight.contains("Key_2")){
            tWeightText = settWeight.getString("Key_2",null);
        } else {
            tWeightText = null;
        }
        tWeight.setText(tWeightText);

        //CHECK IF TARGET HAS BEEN REACHED
        int myTargetNum = 0;
        int myCurrentNum = 0;
        String CongMessage = "You Have reached your Target Weight, Congratulations!!";
        try {
            myTargetNum = Integer.parseInt(tWeightText);
        } catch(NumberFormatException nfe) {
            System.out.println("Could not parse " + nfe);
        }
        try {
            myCurrentNum = Integer.parseInt(cWeightText);
        } catch(NumberFormatException nfe) {
            System.out.println("Could not parse " + nfe);
        }

        //GET CELL NUMBER
        SharedPreferences getCell = getSharedPreferences("myPref",0);
        if(getCell.contains("Key_3")){
            sCellNumber = getCell.getString("Key_3",null);
        } else {
            sCellNumber = "5555555555";
        }

        //CHECK IF TARGET HAS BEEN MET, IF SO SEND TEXT MESSAGE
        if(myCurrentNum==myTargetNum){
            SmsManager mySMS = SmsManager.getDefault();
            mySMS.sendTextMessage(sCellNumber,null,CongMessage,null,null);
            toastMessage("You have Reached Your Target Goal");
            toastMessage("A Text Message has been sent");
        }

        //SET "ROAD TO TARGET" DISPLAY
        Weightdb = new DataBaseWeight(this);
        data = Weightdb.getListContents();
        weightHistory = new StringBuilder();
        if(data.getCount() == 0){
            mScrollText.setText("You have No Weight History");
        }
        else{
            while (data.moveToNext()){
                weightHistory.append("\n" + data.getString(3) + " - " + data.getString(2) + " lbs" );
                mScrollText.setText(weightHistory);
            }
        }

        //CALL SET TARGET SCREEN
        eTargetButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                openEnterTargetWeight();
            }
        });

        //CALL SET CURRENT SCREEN
        cWeightButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                openEnterCurrentWeight();
            }
        });

        //CALL SEE HISTORY SCREEN
        tHistory.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                OpenHistoricalData();
            }
        });

        //CALL SIGN OUT
        tSignOut.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                SignOut();
            }
        });
    }

    //METHOD TO OPEN ENTER TARGET
    public void openEnterTargetWeight(){
        Intent intent = new Intent (this, EnterTargetWeight.class);
        startActivity(intent);
    }

    //METHOD TO OPEN ENTER CURRENT
    public void openEnterCurrentWeight(){
        Intent intent = new Intent (this, EnterCurrentWeight.class);
        startActivity(intent);
    }

    //METHOD TO OPEN HISTORY
    public void OpenHistoricalData(){
        Intent intent = new Intent (this, HistoricalData.class);
        startActivity(intent);
    }

    //METHOD TO SIGNOUT
    public void SignOut(){
        Intent intent = new Intent (this, LoginScreen.class);
        startActivity(intent);
    }

    //TOAST METHOD
    private void toastMessage(String message){
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
}
```
