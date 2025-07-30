# 7Digital Webhook Integration

## Webhook Flow:
**7Digital → POST /synchronize-app-data → Queue → Database**

---

## 1. Endpoint Details:
**File:** `src/routes/webhook7digital/webhook.routes.js`

```js
routes.post(
  '/synchronize-app-data',
  authorizeRequest,
  webhookController.synchronize7DigitalData
);
```

---

## 2. Processing Steps:

### A) Webhook Data Reception:
```js
export const synchronize7DigitalData = async (req, res, next) => {
  try {
    const webhookData = req.body[0];  // Data received from 7Digital
    await queue.add({ webhookData });  // Adding to queue
    return res.status(200).json({ message: 'Data synchronized successfully' });
  } catch (error) {
    console.error(error);
    return res.status(500).json({ message: 'Internal server error' });
  }
};
```

### B) Queue Initialization & Processing:
```js
import Bull from 'bull';

const queue = new Bull('webhookQueue_1', {
  redis: {
    host: 'redis-18476.fcrce180.us-east-1-1.ec2.redns.redis-cloud.com',
    port: 18476,
    password: 'nCEvvQeaONdJVHzzSfdZyflGJyIRyTFf'
  }
});

queue.process(30, async (job) => {
  console.log('Processing Webhook Data!');
  const { webhookData } = job.data;

  if (webhookData.takedown) {
    await updateTakeDownRelease(webhookData.id);
  } else {
    await processWebhookData(webhookData);
  }
});

queue.on('connect', () => {
  console.log('Connected to Redis');
});

queue.on('failed', (job, error) => {
  console.error(`Job ${job.id} failed with error: ${error.message}`);
});
```

### C) Artist Data Processing:
```js
async function processWebhookData(webhookData) {
  const [existingRelease, labelData, artistData] = await Promise.all([
    DataServices.findOne(Release, { id: webhookData.id }),
    updateLabel(webhookData.label.id, webhookData.label),
    updateDataModelWise(ArtistModel, webhookData.artist.id, webhookData.artist)  // Artist create/update
  ]);
  // ... release and tracks processing
}
```

---

## Trigger Conditions (Real-time):
✅ When a new artist is added  
✅ When an existing artist is updated  
✅ When an artist is deleted  
✅ When a release is updated  

---

## Fully Automated Process:
1. A change occurs in 7Digital  
2. 7Digital sends a webhook  
3. `/synchronize-app-data` endpoint receives the data  
4. Job is added to the queue  
5. Data is processed in the background  
6. Artist is created/updated in the database  

---

## Benefits:
✅ **Fully Automated** – No manual intervention needed  
✅ **Real-time** – Immediate sync  
✅ **Reliable** – Uses a queue system  
✅ **Scalable** – Can process 30 concurrent jobs  

---

So yes, whenever a new artist is added in 7Digital, the `/synchronize-app-data` endpoint is triggered automatically!
