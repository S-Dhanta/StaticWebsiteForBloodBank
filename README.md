# Static Website For Blood Bank on cloud

A static website for managing blood donors, hosted on AWS S3, and designed to display and add donor information. Users can search for donors by blood group or register as a donor. Donor data is stored in a JSON file, `donors.json`, located in the `donor-details` folder.

## Project Structure

- **index.html**: Main webpage for viewing and registering donors.
- **donor-details/donors.json**: JSON file containing registered donor details.

## Features

- **Donor Registration Form**: Users can submit their name, blood group, phone number, city, and state to register as a donor.
- **Donor Lookup**: Allows filtering donors by blood group.

## Hosting and Configuration

This project uses **AWS S3**, **Lambda**, and **API Gateway**.

1. **AWS S3**: Hosts the static website files (`index.html` and `donors.json`).
2. **AWS Lambda**: Processes new donor registrations and updates `donors.json`.
3. **AWS API Gateway**: Acts as the endpoint for the Lambda function, handling POST requests from the registration form.

### Setup Instructions

1. **Upload Files to S3**:
   - Upload `index.html` and `donor-details/donors.json` to an S3 bucket.
   - Enable static website hosting on the S3 bucket.
   - Update the bucket policy to allow public read access.

2. **Configure Lambda**:
   - Create a Lambda function that updates `donors.json` with new donor information.
   - Assign the necessary IAM role with permissions to read and write to the S3 bucket.

3. **API Gateway**:
   - Create an HTTP API that triggers the Lambda function on POST requests.
   - Configure CORS to allow requests from your S3 bucket's website URL.

## Lambda Code

Below is the Lambda function code for handling donor registration and saving the data to `donors.json` on S3:

```javascript
import { S3Client, GetObjectCommand, PutObjectCommand } from "@aws-sdk/client-s3"; // Use specific commands for SDK v3

// Initialize the S3 client
const s3 = new S3Client({ region: 'your-region' }); // replace 'your-region' with your AWS region
const bucketName = 'your-bucket-name'; // replace with your S3 bucket name
const fileName = 'donor-details/donors.json'; // replace with the path to your donors.json

export const handler = async (event) => { 
    // Log incoming event
    console.log('Received event:', JSON.stringify(event, null, 2));

    // Parse the incoming request body
    let donorData;
    try {
        donorData = JSON.parse(event.body);
    } catch (error) {
        console.error('JSON Parsing Error:', error);
        return {
            statusCode: 400,
            body: JSON.stringify({ message: 'Invalid JSON' }),
        };
    }

    try {
        // Get existing donors from S3
        const getObjectCommand = new GetObjectCommand({ Bucket: bucketName, Key: fileName });
        const existingData = await s3.send(getObjectCommand);

        const donors = JSON.parse(await streamToString(existingData.Body));

        // Add the new donor to the list
        donors.push(donorData);

        // Save the updated list back to S3
        const putObjectCommand = new PutObjectCommand({
            Bucket: bucketName,
            Key: fileName,
            Body: JSON.stringify(donors, null, 2),
            ContentType: 'application/json'
        });
        await s3.send(putObjectCommand);

        return {
            statusCode: 200,
            body: JSON.stringify({ message: 'Donor information updated successfully.' }),
        };
    } catch (error) {
        console.error('Error updating donor information:', error);
        return {
            statusCode: 500,
            body: JSON.stringify({ message: 'Failed to update donor information (Lambda).', error: error.message }),
        };
    }
};

// Helper function to convert a readable stream to a string
const streamToString = (stream) => {
    return new Promise((resolve, reject) => {
        const chunks = [];
        stream.on("data", (chunk) => chunks.push(chunk));
        stream.on("error", reject);
        stream.on("end", () => resolve(Buffer.concat(chunks).toString("utf-8")));
    });
};
```
This Lambda function code includes the main functionality for:

1. Parsing the incoming HTTP POST request to retrieve donor information.
2. Fetching existing donor data from `donors.json` stored in the S3 bucket.
3. Appending the new donor data to the existing list.
4. Writing the updated list back to `donors.json` in the S3 bucket.

### Endpoints

- **GET `/donors`**: Retrieves the list of available blood donors from `donors.json` stored in S3.
- **POST `/donors-details`**: Accepts a new donor's data and updates the `donors.json` file in S3.

### Important Notes

- Replace `'your-region'` and `'your-bucket-name'` in the Lambda function code with your specific AWS region and S3 bucket name.
- Ensure that your Lambda function has permissions to access and modify objects in your S3 bucket. Use an IAM role with appropriate S3 permissions attached to the Lambda function.

## How to Deploy

1. **AWS S3**: Upload `index.html` and the `donors.json` file (inside the `donor-details` folder) to an S3 bucket.
   - Enable static website hosting on the S3 bucket.
   - Configure CORS settings on the bucket to allow requests from the frontend.

2. **AWS Lambda**: Deploy the Lambda function in the AWS console or using the AWS CLI.
   - Set up the Lambda function as shown in the code, with the necessary environment variables (bucket name, region).
   
3. **API Gateway**: Configure API Gateway to serve as a REST API front for the Lambda function.
   - Create routes for `POST /donors-details` and integrate with the Lambda function.
   - Set up **CORS** on API Gateway for cross-origin requests from your frontend.

## API Usage

- **Endpoint**: `https://<api-id>.execute-api.<region>.amazonaws.com/<stage>/donors-details`
- **Method**: `POST`
- **Request Body**:
    ```json
    {
      "name": "John Doe",
      "bloodGroup": "A+",
      "phone": "1234567890",
      "city": "Sample City",
      "state": "Sample State"
    }
    ```

## Usage

Once deployed, users can:

- **View available blood donors** by selecting a blood group on the main page.
- **Register as a new donor** by filling out the registration form, which sends the data to the backend via the API Gateway and updates the S3 `donors.json` file.

## Troubleshooting

- **Data not updating**: Check Lambda function logs in CloudWatch to ensure there are no errors. Verify S3 permissions.
- **CORS errors**: Ensure CORS is enabled on both S3 and API Gateway.
- **File size increases without visible changes**: Check the Lambda function for unnecessary append operations or log handling issues.

## Acknowledgments

- AWS S3 for static hosting.
- AWS Lambda and API Gateway for serverless backend integration.



---

## License

This project is open-source and available for use and modification.

---

NOTE : This README covers the essentials for setting up, running, and troubleshooting the Blood Bank Project.
