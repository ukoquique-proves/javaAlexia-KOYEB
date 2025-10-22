# Deploying Alexia to Koyeb

This guide provides step-by-step instructions for deploying the Alexia application to Koyeb using Docker and connecting it to a Supabase database.

## Prerequisites

1. **GitHub Repository**: The project code must be pushed to a GitHub repository.
2. **Koyeb Account**: A free account on [Koyeb](https://www.koyeb.com/).
3. **Supabase Account**: A Supabase project with a PostgreSQL database. You will need your database connection details (URL, user, and password).

---

## Deployment Steps

### Step 1: Create a New App on Koyeb

1. Log in to your Koyeb account.
2. On the **Overview** page, click **Create App**.

### Step 2: Choose the Deployment Method

1. Select **GitHub** as the deployment method.
2. If you haven't already, install the Koyeb GitHub App and grant it access to your repository.
3. Choose your project repository from the list.

### Step 3: Configure the Service

Koyeb will automatically detect the `Dockerfile` in your repository.

1. **Deployment Method**: Ensure **Dockerfile** is selected.
2. **App and Service Names**: You can keep the default names or change them (e.g., `alexia-app`).
3. **Port**: Koyeb automatically detects the `EXPOSE` instruction from the `Dockerfile`. It should be set to `8080`.

### Step 4: Add Environment Variables

This is the most important step for connecting to your Supabase database and configuring the application.

1. Scroll down to the **Environment variables** section.
2. Click **Add Variable** and add the following variables. Make sure to select the **Secret** type for sensitive values.

| Name | Type | Value |
| :--- | :--- | :--- |
| `SPRING_DATASOURCE_URL` | `Secret` | `jdbc:postgresql://db.hgcesbylhkjoxtymxysy.supabase.co:6543/postgres?sslmode=disable&prepareThreshold=0` |
| `SPRING_DATASOURCE_USERNAME` | `Value` | `postgres` |
| `SPRING_DATASOURCE_PASSWORD` | `Secret` | `<YOUR_SUPABASE_DB_PASSWORD>` |
| `TELEGRAM_BOT_TOKEN` | `Secret` | `<YOUR_TELEGRAM_BOT_TOKEN>` |
| `TELEGRAM_BOT_USERNAME` | `Value` | `<YOUR_BOT_USERNAME>` |
| `GROK_API_KEY` | `Secret` | `<YOUR_GROK_API_KEY>` |
| `GROK_MODEL` | `Value` | `llama-3.1-8b-instant` |
| `GROK_API_URL` | `Value` | `https://api.groq.com/openai/v1/chat/completions` |

**Where to find these values?**
- **Supabase**: Go to your Supabase project → **Project Settings** → **Database**.
  - Under **Connection string**, you will find the Host, Database name, and User.
  - Use port `6543` (connection pooler) instead of `5432` for better performance.
- **Telegram**: Get your bot token from [@BotFather](https://t.me/botfather) on Telegram.
- **Grok**: Get your API key from [Groq Console](https://console.groq.com/).

**Optional Environment Variables** (for future features):
- `GOOGLE_PLACES_API_KEY` - For external business search
- `WHATSAPP_API_KEY` - For WhatsApp Business integration
- `OPENAI_API_KEY` - For OpenAI integration

### Step 5: Deploy

1. Click the **Deploy** button.

Koyeb will now start building your application from the `Dockerfile` and deploy it. The first build may take 5-10 minutes because:
- Maven downloads all dependencies
- Vaadin builds the production frontend bundle
- Docker creates and pushes the container image

Once the deployment is complete, Koyeb will provide a public URL (e.g., `https://alexia-app-org.koyeb.app`) where you can access your live application.

---

## Post-Deployment Configuration

### 1. Set Telegram Webhook (Optional)

If you want to use webhooks instead of polling for Telegram updates:

```bash
curl -X POST "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=https://your-app.koyeb.app/webhooks/telegram"
```

### 2. Test the Application

1. Access the dashboard at `https://your-app.koyeb.app`
2. Test the Telegram bot by sending a message
3. Check the logs in Koyeb dashboard for any errors

---

## Troubleshooting

### Build Takes Too Long
The first build is slow because Vaadin needs to compile the frontend. Subsequent builds will be faster (2-3 minutes).

### Application Shows 500 Error
Ensure that:
1. `vaadin.productionMode=true` is set in `application.properties`
2. The `vaadin-maven-plugin` is configured in `pom.xml`
3. All environment variables are correctly set
4. Database connection string includes `?sslmode=disable&prepareThreshold=0`

### Database Connection Errors
If you see "prepared statement already exists" errors:
- Make sure you're using port `6543` (connection pooler) not `5432`
- Ensure `prepareThreshold=0` is in the connection string
- Check that `sslmode=disable` is set for the pooler

### Port Mismatch
The application is configured to run on port **8080**. Koyeb should automatically detect this from the `Dockerfile`. If you see port-related errors, verify that:
- `server.port=8080` in `application.properties`
- `EXPOSE 8080` in `Dockerfile`

### Telegram Bot Not Responding
1. Check that `TELEGRAM_BOT_TOKEN` is correctly set
2. Verify the bot is not running elsewhere (409 Conflict error)
3. Check application logs in Koyeb dashboard

---

## Monitoring and Logs

### View Logs
1. Go to your Koyeb dashboard
2. Select your app
3. Click on the **Logs** tab
4. Filter by severity (Info, Warning, Error)

### Health Checks
Koyeb automatically monitors your application health. If the application becomes unresponsive, Koyeb will restart it automatically.

---

## Updating the Application

### Automatic Deployments
Koyeb is configured to automatically deploy when you push to your GitHub repository:

1. Make changes to your code
2. Commit and push to GitHub
3. Koyeb will automatically detect the changes and redeploy

### Manual Redeploy
1. Go to your Koyeb dashboard
2. Select your app
3. Click **Redeploy**

---

## Cost Considerations

### Free Tier Limits
Koyeb offers a free tier with:
- 1 web service
- 512 MB RAM
- 2 GB disk
- Shared CPU

This is sufficient for development and small-scale production use.

### Upgrading
If you need more resources:
- Upgrade to a paid plan for dedicated CPU and more RAM
- Consider using Koyeb's auto-scaling features

---

## Comparison with Render

| Feature | Koyeb | Render |
|---------|-------|--------|
| Free Tier | ✅ 512 MB RAM | ✅ 512 MB RAM |
| Auto-deploy from GitHub | ✅ Yes | ✅ Yes |
| Docker Support | ✅ Native | ✅ Native |
| Build Time | ~5-10 min | ~5-10 min |
| Cold Start | Fast | Slow (free tier) |
| Custom Domains | ✅ Yes | ✅ Yes |
| Environment Variables | ✅ Secrets support | ✅ Secrets support |

**Recommendation**: Koyeb has faster cold starts on the free tier, making it better for development and testing.

---

## Success!

Your Alexia application is now deployed on Koyeb and connected to your Supabase database. You can:
- Access the dashboard at your Koyeb URL
- Chat with the Telegram bot
- Monitor logs and metrics in the Koyeb dashboard
- Set up automatic deployments from GitHub

For production use, consider:
- Setting up a custom domain
- Enabling HTTPS (automatic with Koyeb)
- Configuring monitoring and alerts
- Setting up database backups in Supabase
