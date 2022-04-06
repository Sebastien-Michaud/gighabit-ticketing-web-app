<script setup>
import { ref } from 'vue'
import { DynamoDB, DynamoDBClient, UpdateItemCommand } from "@aws-sdk/client-dynamodb"
import { PutCommand, QueryCommand } from "@aws-sdk/lib-dynamodb"
import axios from "axios"

// Set the AWS Region.
const REGION = "eu-west-2" // For example, "us-east-1".

//Set credentials


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

  submitted.value = true
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

  <p>Submitted: {{ submitted }}</p>
  <p>Log: {{ log }}</p>
  <p>ID: {{ ID }}</p>
  <p>tableData: {{ tableData }}</p>
  <p>originalID: {{ originalID }}</p>
  <p>token: {{ token }}</p>
  <p>REST data: {{ data }}</p>
</template>
