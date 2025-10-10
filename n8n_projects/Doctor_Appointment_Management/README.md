# Doctor Appointment Management System

A comprehensive AI-powered appointment booking system built with n8n that enables patients to book, manage, and pay for doctor appointments via WhatsApp. The system integrates with Google Sheets for data storage, Stripe for payment processing, and uses Google Gemini AI for intelligent conversational interactions.

![System Overview](screenshots/system-overview.jpg)

## 📋 Table of Contents

- [Features](#features)
- [System Architecture](#system-architecture)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Workflow Components](#workflow-components)
- [User Flow](#user-flow)
- [Database Structure](#database-structure)
- [Configuration](#configuration)
- [Usage Examples](#usage-examples)
- [API Integrations](#api-integrations)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## ✨ Features

### Core Functionality
- **WhatsApp Integration**: Complete appointment management through WhatsApp messaging
- **AI-Powered Assistant**: Natural language processing using Google Gemini for intelligent conversations
- **Multi-Patient Support**: Single users can register and manage multiple patients
- **Smart Scheduling**: Automatic slot management with conflict prevention
- **Payment Processing**: Integrated Stripe payment with automatic payment link generation
- **Automated Reminders**: Daily appointment reminders sent automatically
- **Refund Management**: Automatic refund processing for cancelled appointments

### Appointment Features
- 📅 **New Booking**: Step-by-step appointment creation
- 📋 **View Bookings**: List all upcoming appointments
- 🔄 **Reschedule**: Modify existing appointment dates/times
- ❌ **Cancel**: Cancel appointments with automatic refund (if paid)
- 💳 **Flexible Payment**: Stripe online payment or cash at clinic
- 📱 **Real-time Updates**: Instant confirmation messages

![WhatsApp Chat Interface](screenshots/whatsapp-interface.jpg)

## 🏗️ System Architecture

The system consists of five main workflow sections:

### 1. Appointment Management Workflow
Handles the core booking, viewing, rescheduling, and cancellation operations through WhatsApp.


### 2. Payment Link Generation
Automatically generates and sends Stripe payment links when online payment is selected.

![Payment Link Generation](screenshots/payment-link-workflow.jpg)

### 3. Payment Confirmation
Processes successful Stripe payments and updates appointment status.

![Payment Confirmation Workflow](screenshots/payment-link-workflow.jpg)

### 4. Refund Processing
Handles automatic refunds and status updates for cancelled paid appointments.


### 5. Appointment Reminders
Scheduled daily reminders sent to patients with confirmed appointments.

![Appointment Reminders Workflow](screenshots/reminder-workflow.jpg)

## 📦 Prerequisites

### Required Services & Accounts

1. **n8n Installation**
   - Self-hosted n8n instance or n8n Cloud account
   - Version: Latest stable release

2. **WhatsApp Business Account**
   - WhatsApp Business API access
   - Verified phone number
   - API credentials

3. **Google Account**
   - Google Sheets API enabled
   - OAuth2 credentials configured

4. **Stripe Account**
   - Active Stripe account (Test/Live mode)
   - API keys (Publishable and Secret)
   - Webhook endpoint configured

5. **Google Gemini API**
   - API key from Google AI Studio
   - Enabled API access

![Prerequisites Setup](screenshots/prerequisites.png)

## 🚀 Installation & Setup

### Step 1: Import Workflow to n8n

1. Open your n8n instance
2. Click on **Workflows** → **Import from File**
3. Select the `Doctor Appointment.json` file
4. Click **Import**

![Import Workflow](screenshots/import-workflow.png)

### Step 2: Configure Google Sheets

1. Create a new Google Spreadsheet named "Doctor Appointment"
2. Create three sheets with the following structure:

#### **Patients Sheet**
| Column | Description |
|--------|-------------|
| patient_id | Unique identifier (auto-generated) |
| whatsapp_number | Patient's WhatsApp number |
| name | Patient's full name |
| age | Patient's age |
| gender | Patient's gender |

#### **Appointments Sheet**
| Column | Description |
|--------|-------------|
| appointment_id | Unique identifier (auto-generated) |
| patient_id | Reference to patient |
| whatsapp_number | Contact WhatsApp number |
| date | Appointment date (YYYY-MM-DD) |
| time | Appointment time (HH:MM) |
| payment_method | Stripe/Cash at Clinic |
| payment_status | Pending/Paid/Refunded |
| status | Confirmed/Cancelled |
| stripe_payment_intent | Stripe payment intent ID |

#### **Config Sheet**
| key | value |
|-----|-------|
| working_hours | 10:00-18:00 |
| not_available | YYYY-MM-DD HH:MM to HH:MM |

3. Note the Spreadsheet ID from the URL
4. Update all Google Sheets nodes with your Spreadsheet ID

![Google Sheets Structure](screenshots/google-sheets-structure.png)

### Step 3: Configure Credentials

#### WhatsApp Credentials
1. Open any WhatsApp node
2. Click **Create New Credential**
3. Enter:
   - Phone Number ID
   - Access Token
   - Webhook verification token

![WhatsApp Credentials](screenshots/whatsapp-credentials.png)

#### Google Sheets OAuth2
1. Open any Google Sheets node
2. Click **Create New Credential**
3. Follow OAuth2 authentication flow
4. Grant necessary permissions

![Google Sheets OAuth](screenshots/google-oauth.png)

#### Stripe API
1. Navigate to Stripe nodes
2. Update the Authorization header in HTTP Request nodes:
   ```
   Bearer YOUR_STRIPE_SECRET_KEY
   ```
3. Configure Stripe Trigger webhook

![Stripe Configuration](screenshots/stripe-config.png)

#### Google Gemini API
1. Open Google Gemini Chat Model nodes
2. Add your API key
3. Configure model settings (optional)

![Gemini Configuration](screenshots/gemini-config.png)

### Step 4: Configure Webhooks

1. **WhatsApp Webhook**
   - Copy webhook URL from WhatsApp Trigger node
   - Configure in WhatsApp Business API settings

2. **Stripe Webhook**
   - Copy webhook URL from Stripe Trigger node
   - Add to Stripe Dashboard webhooks
   - Select event: `payment_intent.succeeded`

![Webhook Configuration](screenshots/webhook-config.png)

### Step 5: Update Timezone & Settings

1. Open **Date & Time** nodes
2. Set timezone to your location (currently: Asia/Karachi)
3. Configure **Schedule Trigger** for appointment reminders
   - Default: Daily at 16:17
   - Adjust as needed

![Timezone Configuration](screenshots/timezone-config.png)

### Step 6: Test the Workflow

1. Activate all workflows
2. Send "Hi" to your WhatsApp Business number
3. Follow the booking process
4. Verify data is stored in Google Sheets

![Testing Workflow](screenshots/testing.png)

## 🔧 Workflow Components

### Node Breakdown

#### **1. WhatsApp Trigger**
- Listens for incoming WhatsApp messages
- Triggers the main AI Agent workflow
- Extracts message content and sender information

#### **2. AI Agent**
- Core conversational AI powered by Google Gemini
- Manages conversation state and context
- Routes users through booking workflow
- Handles natural language queries

#### **3. Simple Memory**
- Maintains conversation context (6-message window)
- Session-based memory using WhatsApp number as key
- Enables stateful conversations

#### **4. Google Sheets Tools**
- **Doctor Config**: Retrieves working hours and availability
- **Get Patient List**: Fetches registered patients for user
- **Get User Appointments**: Retrieves user's appointments
- **Add Appointment**: Creates new appointment records
- **Add Patient**: Registers new patients
- **Get All Appointments**: Fetches complete appointment data
- **Reschedule Appointment**: Updates appointment datetime
- **Cancel Appointment**: Marks appointments as cancelled

#### **5. Date & Time Tool**
- Provides current date/time to AI Agent
- Timezone-aware (Asia/Karachi)
- Used for slot availability calculations

#### **6. Payment Processing Nodes**
- **HTTP Request**: Creates Stripe checkout sessions
- **Send Payment Link**: Sends payment URL via WhatsApp
- **Stripe Trigger**: Listens for successful payments
- **Update Row**: Updates payment status in sheets

#### **7. Refund Workflow Nodes**
- Monitors appointment cancellations
- Processes Stripe refunds automatically
- Updates payment status to "Refunded"
- Sends confirmation messages

#### **8. Reminder System**
- **Schedule Trigger**: Runs daily at specified time
- **Appointment Remainder AI Agent**: Identifies today's appointments
- **Send Message**: Sends reminder via WhatsApp

![Node Architecture](screenshots/node-architecture.png)

## 👥 User Flow

### New Booking Flow

1. **User Initiation**
   ```
   User: Hi
   Bot: Hello, Welcome to Doctor Clinic!
        Please choose an option:
        1. New Booking
        2. My Upcoming Bookings
        3. Reschedule Booking
        4. Cancel Booking
   ```

2. **Patient Selection**
   ```
   User: 1
   Bot: [Shows existing patients or asks to add new]
        1. John Doe (35, Male) - ID: 1234567890
        2. Add New Patient
   ```

3. **Add New Patient** (if selected)
   ```
   Bot: Please provide:
        - Name
        - Age  
        - Gender
   User: Jane Doe, 28, Female
   Bot: Patient added! [Shows updated list]
   ```

4. **Date Selection**
   ```
   Bot: Available dates:
        1. 2025-10-11 (Friday)
        2. 2025-10-12 (Saturday)
        3. 2025-10-13 (Sunday)
        ...
   User: 1
   ```

5. **Time Slot Selection**
   ```
   Bot: Available time slots:
        1. 10:00 - 11:00
        2. 11:00 - 12:00
        3. 13:00 - 14:00
        ...
   User: 2
   ```

6. **Payment Method**
   ```
   Bot: How would you like to pay?
        1. Online (Stripe)
        2. Cash at Clinic
   User: 1
   ```

7. **Confirmation**
   ```
   Bot: Appointment booked!
        Patient: Jane Doe
        Date: 2025-10-12
        Time: 11:00
        Payment Method: Stripe
        Payment Status: Pending
        Status: Confirmed
        
        [Payment link sent separately]
   ```

![Booking Flow Diagram](screenshots/booking-flow.png)

### Viewing Bookings

```
User: 2
Bot: Your upcoming appointments:
     
     1. John Doe
        Date: 2025-10-12
        Time: 11:00
        
     2. Jane Doe
        Date: 2025-10-15
        Time: 14:00
```

### Rescheduling Appointment

```
User: 3
Bot: Select appointment to reschedule:
     
     1. Appointment #1234 - John Doe - 2025-10-12 11:00
     2. Appointment #5678 - Jane Doe - 2025-10-15 14:00

User: 1
Bot: [Shows available dates]
User: [Selects new date]
Bot: [Shows available time slots]
User: [Selects new time]
Bot: Appointment rescheduled successfully!
     New details: 2025-10-14 at 15:00
```

### Cancelling Appointment

```
User: 4
Bot: Select appointment to cancel:
     
     1. Appointment #1234 - 2025-10-12 11:00
     2. Appointment #5678 - 2025-10-15 14:00

User: 1
Bot: Your appointment on 2025-10-12 at 11:00 has been cancelled.
     [If paid via Stripe: Refund has been initiated]
```

![User Flow Diagram](screenshots/user-flow-complete.png)

## 💾 Database Structure

### Entity Relationship

```
┌─────────────┐         ┌──────────────────┐         ┌────────────┐
│  Patients   │────────<│   Appointments   │>────────│   Config   │
└─────────────┘         └──────────────────┘         └────────────┘
     1:N                        1:1                        1:1
                                 │
                                 │
                          ┌──────▼──────┐
                          │   Stripe    │
                          │  Payments   │
                          └─────────────┘
```

### Data Flow

1. **Patient Registration**: WhatsApp number → Patient record
2. **Appointment Creation**: Patient ID + Date/Time → Appointment record
3. **Payment Processing**: Appointment ID → Stripe session → Payment intent
4. **Status Updates**: Stripe webhook → Appointment status update
5. **Refunds**: Cancelled appointment → Stripe refund → Status update

![Database Schema](screenshots/database-schema.png)

## ⚙️ Configuration

### Customizing Working Hours

Edit the Config sheet:
```
key: working_hours
value: 09:00-17:00  (Change as needed)
```

### Setting Doctor Unavailability

Add rows to Config sheet:
```
key: not_available
value: 2025-10-15 10:00 to 12:00
```
This blocks the time slot on that specific date.

### Adjusting Appointment Duration

Currently set to 60-minute slots. To modify:
1. Update the AI Agent system prompt
2. Change: "Divide into slots of 60 minutes" to your desired duration

### Modifying Payment Amount

In the **HTTP Request** node (payment link generation):
```json
{
  "name": "line_items[0][price_data][unit_amount]",
  "value": "5000"  // Amount in cents (50.00 CAD)
}
```

### Changing Reminder Schedule

In the **Schedule Trigger** node:
- Adjust `triggerAtHour` and `triggerAtMinute`
- Current: 16:17 (4:17 PM)

![Configuration Settings](screenshots/configuration.png)

## 📱 Usage Examples

### Example Conversation 1: Quick Booking

```
User: Hi
Bot: Hello, Welcome to Doctor Clinic!
     Please choose an option:
     1. New Booking
     2. My Upcoming Bookings
     3. Reschedule Booking
     4. Cancel Booking

User: 1
Bot: I see you have one registered patient:
     1. Sarah Smith (42, Female) - ID: 1697891234567
     2. Add New Patient
     
     Which one would you like to book for?

User: 1
Bot: Great! Available dates:
     1. 2025-10-11 (Tomorrow)
     2. 2025-10-12 (Saturday)
     3. 2025-10-13 (Sunday)

User: 1
Bot: Available time slots for 2025-10-11:
     1. 10:00 - 11:00
     2. 13:00 - 14:00
     3. 15:00 - 16:00

User: 2
Bot: How would you like to pay?
     1. Online (Stripe)
     2. Cash at Clinic

User: 2
Bot: Appointment booked!
     Patient: Sarah Smith
     Date: 2025-10-11
     Time: 13:00
     Payment Method: Cash at Clinic
     Payment Status: Pending
     Status: Confirmed
```

### Example Conversation 2: Adding New Patient

```
User: Book appointment
Bot: I don't see any patients registered with your number.
     Let's add a new patient.
     
     Please provide:
     - Name
     - Age
     - Gender

User: Michael Johnson, 56, Male
Bot: Patient added successfully!
     
     Your patients:
     1. Michael Johnson (56, Male) - ID: 1697891234890
     
     Now, let's book the appointment...
     [Continues with date selection]
```

![Usage Examples](screenshots/usage-examples.png)

## 🔌 API Integrations

### WhatsApp Business API

**Configuration:**
- Phone Number ID: Set in WhatsApp nodes
- Access Token: Required for sending messages
- Webhook URL: For receiving incoming messages

**Endpoints Used:**
- Send Text Message
- Webhook for receiving messages

### Stripe API

**Configuration:**
- Secret Key: `sk_test_...` (Replace with your key)
- Webhook Secret: For payment verification

**Endpoints Used:**
- `POST /v1/checkout/sessions` - Create payment session
- `GET /v1/checkout/sessions` - Retrieve session details
- `POST /v1/refunds` - Process refunds
- Webhook: `payment_intent.succeeded` event

**Security Note:** Store Stripe keys in n8n credentials, not directly in nodes.

### Google Sheets API

**Configuration:**
- OAuth2 authentication
- Spreadsheet ID
- Sheet names (Patients, Appointments, Config)

**Operations Used:**
- Read rows (filtered and unfiltered)
- Append rows
- Update rows

### Google Gemini AI API

**Configuration:**
- API Key from Google AI Studio
- Model: Gemini Pro (default)

**Usage:**
- Natural language understanding
- Conversation flow management
- Context-aware responses


## 🐛 Troubleshooting

### Common Issues

#### 1. AI Agent Not Responding
**Symptoms:** No response after sending WhatsApp message

**Solutions:**
- Check WhatsApp Trigger is activated
- Verify webhook URL in WhatsApp Business settings
- Ensure Google Gemini API key is valid
- Check n8n execution logs for errors

#### 2. Payment Link Not Generated
**Symptoms:** No payment link received after selecting Stripe payment

**Solutions:**
- Verify Stripe API key is correct
- Check "If" node condition (payment_method contains "Stripe")
- Ensure Google Sheets Trigger is monitoring new rows
- Verify appointment was saved to Appointments sheet

#### 3. Slots Not Showing Correctly
**Symptoms:** Booked slots still appear as available

**Solutions:**
- Check "Get All Appointments" tool is connected to AI Agent
- Verify date format in Appointments sheet (YYYY-MM-DD)
- Ensure time format is consistent (HH:MM)
- Review AI Agent prompt for slot exclusion logic

#### 4. Refunds Not Processing
**Symptoms:** Cancellation confirmed but no refund initiated

**Solutions:**
- Verify `stripe_payment_intent` field has value
- Check Stripe API key permissions include refunds
- Review "If2" node condition for empty check
- Check HTTP Request2 node headers and body

#### 5. Reminders Not Sending
**Symptoms:** No reminder messages sent for today's appointments

**Solutions:**
- Verify Schedule Trigger is activated
- Check timezone setting matches your location
- Ensure appointments have status "Confirmed" or "Pending"
- Review Appointment Remainder AI Agent prompt

#### 6. Memory Issues
**Symptoms:** Bot doesn't remember previous conversation

**Solutions:**
- Check Simple Memory node session key configuration
- Verify context window length (currently 6 messages)
- Test with fresh conversation after making changes

### Debug Mode

Enable debug mode in n8n:
1. Go to Settings → Environments
2. Set `EXECUTIONS_DATA_SAVE_ON_ERROR=all`
3. Set `EXECUTIONS_DATA_SAVE_ON_SUCCESS=all`
4. Review execution logs for detailed error messages



## 🧪 Testing Checklist

- [ ] WhatsApp message triggers workflow
- [ ] Main menu displays correctly
- [ ] Patient registration works
- [ ] Existing patients load correctly
- [ ] Date selection shows next 7 days
- [ ] Fully booked dates are excluded
- [ ] Time slots respect working hours
- [ ] Booked slots are excluded
- [ ] Today's slots only show future times
- [ ] Payment method selection works
- [ ] Stripe payment link is generated
- [ ] Payment link arrives via WhatsApp
- [ ] Payment confirmation updates status
- [ ] Cash payment records correctly
- [ ] View bookings shows correct data
- [ ] Reschedule finds user appointments
- [ ] Reschedule updates date/time correctly
- [ ] Cancel marks appointment as cancelled
- [ ] Refund processes for paid cancellations
- [ ] Reminder sends for today's appointments
- [ ] Conversation memory persists across messages



## 🤝 Contributing

Contributions are welcome! Here's how you can help:

### Reporting Issues
1. Check existing issues first
2. Provide detailed description
3. Include n8n version and error logs
4. Share workflow execution screenshots

### Feature Requests
1. Describe the feature clearly
2. Explain use case and benefits
3. Suggest implementation approach

### Code Contributions
1. Fork the repository
2. Create a feature branch
3. Test thoroughly
4. Submit pull request with description

### Areas for Enhancement
- Multi-language support
- SMS integration as backup
- Email notifications
- Patient medical history tracking
- Doctor availability patterns
- Appointment analytics dashboard
- Video consultation links
- Insurance verification


## 📄 License

This project is licensed under the MIT License.

```
MIT License

Copyright (c) 2025 Doctor Appointment System

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```


## 🙏 Acknowledgments

- **n8n** - Workflow automation platform
- **Google Gemini AI** - Conversational AI capabilities
- **Stripe** - Payment processing
- **WhatsApp Business** - Messaging platform
- **Google Sheets** - Data storage

---

**Built with ❤️ using n8n By PforProgrammer**

Last Updated: October 10, 2025
Version: 1.0.0

