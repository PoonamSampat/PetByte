## Pet Monitor – Pet Bytes
During the development of the Pet Bytes idea, we had two challenges:
1.           Ingesting messages from IoT Devices into Azure.
2.           When the sound signals captured from the device reached a certain threshold a signal had to be sent to the bot framework conveying the same with a picture taken of the pet at the time when the sound signals were different than normal.


### Case 1:       
We used Stream Analytics service to ingest the messages.
Here is the query written to do the same:

```sql
SELECT
Temperature as BodyTemperature,DeviceId as PID,TimeStamp1 as SensorDateTime,Sound as PetSound 
INTO
OutputData
FROM
InputData
```

### Case 2:
We used functions to solve case 2 by sending a message to the bot framework when there was a spike in the sound signal.

```cs
using System.Data;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;


public static void Run(TimerInfo myTimer, TraceWriter log)
{
    System.Data.SqlClient.SqlConnection sqlConnection1 =
            new System.Data.SqlClient.SqlConnection("Data Source=fbpetserver.database.windows.net;Initial Catalog=FBPetStats;Persist Security Info=True;User ID=petadmin;Password=W@lcome123");
    
    var stmt = "Select Count(PID) as sensorcount from PetSensorData where PetSound > 35  and SensorDateTime > DATEADD(SECOND, -30,  GETUTCDATE())";

            System.Data.SqlClient.SqlCommand cmd = new       System.Data.SqlClient.SqlCommand();
            cmd.CommandType = System.Data.CommandType.Text;
            cmd.CommandText = stmt;
            cmd.Connection = sqlConnection1;
            System.Data.SqlClient.SqlDataReader reader;
            sqlConnection1.Open();
            reader = cmd.ExecuteReader();
            if (reader.HasRows)
            {
                while (reader.Read())
                {
                string column = reader["sensorcount"].ToString();
                int columnValue = Convert.ToInt32(reader["sensorcount"]);
                if (columnValue > 0)
                    {
                    string urlWithAccessToken = "https://hooks.slack.com/services/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";
                    System.Net.Http.HttpClient client = new System.Net.Http.HttpClient();
                    var content = "{\"text\": \">>>Tom seems to be barking loudly\nShow me Tom.\"}";
                        var result = client.PostAsync(new Uri(urlWithAccessToken), new System.Net.Http.StringContent(content)).Result;        
                  }}
sqlConnection1.Close();
} 
```

We used SQL Database to store the messages.
Json template for the same:
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "servers_fbpetserver_name": {
            "value": null
        },
        "databases_FBPetStats_name": {
            "value": null
        },
        "databases_master_name": {
            "value": null
        },
        "firewallRules_Allconnection_name": {
            "value": null
        },
        "firewallRules_AllowAllWindowsAzureIps_name": {
            "value": null
        }
    }
}
```






