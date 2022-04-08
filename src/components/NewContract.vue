<script setup>
import { ref } from "vue"
import { DynamoDB, DynamoDBClient, UpdateItemCommand } from "@aws-sdk/client-dynamodb"
import { PutCommand, QueryCommand } from "@aws-sdk/lib-dynamodb"
import AWSCredentials from "../credentials/AWSCredentials.json"

// Set the AWS Region.
const REGION = "eu-west-2" 

//Set credentials
const accessKeyId = AWSCredentials['accessKeyId']
const secretAccessKey = AWSCredentials['secretAccessKey']

// Create an Amazon DynamoDB service client object.
const ddbClient = new DynamoDBClient({ credentials: { accessKeyId: accessKeyId, secretAccessKey: secretAccessKey }, region: REGION })

const eventName = ref('')
const eventStartDate = ref('')
const eventEndDate = ref('')

const submitted = ref(false)
const log = ref('')
const ID = ref('')
const originalID = ref('')
const tableData = ref('')

const signOutURL = "https://gighabit-ticketing.auth.eu-west-2.amazoncognito.com/logout?client_id= kelm8fid312g68ul7h1ml9f9q&logout_uri=https://gighabit-ticketing.auth.eu-west-2.amazoncognito.com/login?client_id=kelm8fid312g68ul7h1ml9f9q&response_type=code&scope=email+openid+phone&redirect_uri=https%3A%2F%2F18.130.254.119%3A3000%2F"

async function handleSubmit() {
  
  //Get contract ID
  const queryCommandParams = {
    KeyConditionExpression: "id = :i",
    ExpressionAttributeValues: {
      ":i": "currentID"
    },
    TableName: "gighabit_ticketing_smart_contracts",
  }
  try {
    tableData.value = await ddbClient.send(new QueryCommand(queryCommandParams))
    originalID.value = tableData.value['Items'][0]['currentID'].toString()

    //Define new contract ID
    ID.value = (parseInt(tableData.value['Items'][0]['currentID']) + 1).toString()

  } catch (err) {
    log.value = err
  }

  // Set command params
  const putCommandParams = {
    TableName: "gighabit_ticketing_smart_contracts",
    Item: {
      id: ID.value,
      eventName: eventName.value,
      eventStartDate: eventStartDate.value,
      eventEndDate: eventEndDate.value,
      smartContractCreated: false,
      contractID: ""
    }
  }

  // Send submitted data to dynamoDB table
  try {
    const data = await ddbClient.send(new PutCommand(putCommandParams))
    log.value = "Success - item added or updated, data: " + data
  } catch (err) {
    log.value = err.stack
  }

  //Update max ID
  const updateItemCommandParams = {
    TableName: "gighabit_ticketing_smart_contracts",
    Key: {
        id: { S: 'currentID' }
    },
    UpdateExpression: "set currentID = :c",
    ExpressionAttributeValues: {
        ':c': { S: ID.value },
    },
    ReturnValues: "ALL_NEW"
  }
  try {
    const data = await ddbClient.send(new UpdateItemCommand(updateItemCommandParams))
  } catch (err) {
    log.value = err
  }

  //Empty form fields
  eventName.value = ""
  eventStartDate.value = ""
  eventEndDate.value = ""

  submitted.value = true
}

function logOut(event) {
  //const response = fetch(signOutURL.value)
  //log.value = response
  window.location.href = signOutURL 
}
</script>

<template>
  <h1>Contract</h1>
  
  <form @submit.prevent="handleSubmit">
    <label>
      Name:
      <input v-model="eventName"/>
    </label><br>

    <label>
      Start Date:
      <input v-model="eventStartDate"/>
    </label><br>

    <label>
      End Date:
      <input v-model="eventEndDate"/>
    </label><br>

    <button type="submit">Submit</button>
  </form>
  
  <!--<button @click="logOut">Log Out</button>-->
  
  <form @submit.prevent="logOut"> 
    <button type="submit">Log Out</button>
  </form>
  
  <p>Submitted: {{ submitted }}</p>
  <p>Log: {{ log }}</p>
  <p>ID: {{ ID }}</p>
  <p>tableData: {{ tableData }}</p>
  <p>originalID: {{ originalID }}</p>
  <p>REST data: {{ data }}</p>
</template>
