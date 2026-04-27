# Brand Management - Zapier MCP Embed

## Table of Contents

- [Brand Management - Zapier MCP Embed](#brand-management---zapier-mcp-embed)
  - [Table of Contents](#table-of-contents)
  - [About](#about)
    - [Key Use Cases](#key-use-cases)
    - [How It Works](#how-it-works)
  - [Resources](#resources)
  - [MCP Embed Configuration](#mcp-embed-configuration)
    - [Authorization](#authorization)
  - [Embed Code](#embed-code)
    - [Head Section](#head-section)
    - [Body Section](#body-section)
  - [Available Attributes](#available-attributes)
  - [Quick Account Creation](#quick-account-creation)
    - [Attributes](#attributes)
  - [Event Reference](#event-reference)
    - [mcp-server-url](#mcp-server-url)
    - [tools-changed](#tools-changed)
    - [close-requested](#close-requested)
  - [Integration Flow](#integration-flow)
  - [TypeScript Implementation](#typescript-implementation)
    - [TypeScript Installation](#typescript-installation)
    - [TypeScript Usage](#typescript-usage)
  - [Python Implementation](#python-implementation)
    - [Python Installation](#python-installation)
    - [Python Usage](#python-usage)
  - [WordPress/WooCommerce Integration](#wordpresswoocommerce-integration)
    - [Overview](#overview)
    - [Architecture](#architecture)
    - [Prerequisites](#prerequisites)
    - [Implementation Steps](#implementation-steps)
      - [1. Add PHP Function to Functions.php](#1-add-php-function-to-functionsphp)
      - [2. Create Portal Page Template](#2-create-portal-page-template)
      - [3. Embed Zapier Interface with Authentication Pass-Through](#3-embed-zapier-interface-with-authentication-pass-through)
      - [4. Save MCP Server URL (Optional)](#4-save-mcp-server-url-optional)
    - [Security Considerations](#security-considerations)
    - [Integration Flow Diagram](#integration-flow-diagram)
    - [Architecture Details](#architecture-details)
    - [Benefits](#benefits)
    - [Troubleshooting](#troubleshooting)
  - [Metadata](#metadata)

---

## About

**Zapier MCP Embed** is a web-based integration tool that enables seamless automation across thousands of third-party applications. MCP (Model Context Protocol) provides a standardized interface for connecting to Zapier's automation platform directly from your web application.

### Key Use Cases

- **Workflow Automation**: Allow your users to create automated workflows between different apps without leaving your platform
- **Integration Management**: Enable users to manage their integrations and available tools in one centralized location
- **Quick Account Setup**: Streamline user onboarding by pre-populating signup information
- **Real-time Tool Management**: Dynamically respond when users add or remove tools from their configuration
- **Custom Applications**: Build internal tools that leverage the power of Zapier's 10,000+ connected applications

### How It Works

The embed provides an interactive interface that users can interact with to authenticate with Zapier, select tools and automation workflows, and receive a unique MCP server URL. Your application then uses this URL with the provided secret to connect to the user's MCP server and execute tools on their behalf.

---

## Resources

- [Zapier MCP Embed Script](https://mcp.zapier.com/embed/v1/mcp.js)
- [Zapier Interfaces](https://interfaces.zapier.com)

---

## MCP Embed Configuration

### Authorization

```plaintext
Authorization: Bearer "op://Zapier_MCP/Brand Management/mcp-embed/embed_secret"
```

---

## Embed Code

### Head Section

Add this script tag to your HTML `<head>`:

```html
<script src="https://mcp.zapier.com/embed/v1/mcp.js"></script>
```

### Body Section

Add this to your HTML `<body>`:

```html
<zapier-mcp
  embed-id="op://Zapier_MCP/Brand Management/mcp-embed/embed-id"
  width="100%"
  height="600px"
  sign-up-email="email_of_your_user@example.com"
  sign-up-first-name="first_name_of_your_user"
  sign-up-last-name="last_name_of_your_user"
></zapier-mcp>

<script>
  document
    .querySelector("zapier-mcp")
    .addEventListener("mcp-server-url", (event) => {
      // this is the URL of the MCP server for a specific user
      console.log("MCP Server URL:", event.detail.serverUrl);
    });

  document
    .querySelector("zapier-mcp")
    .addEventListener("tools-changed", (event) => {
      // this is sent whenever a user adds/removes tools from their MCP server
      console.log("Tools changed");
    });

  document
    .querySelector("zapier-mcp")
    .addEventListener("close-requested", (event) => {
      // this is sent when the user clicks the close button in the embed
      console.log("Close requested");
    });
</script>
```

---

## Available Attributes

| Attribute    | Required | Description                                                                                   |
| ------------ | -------- | --------------------------------------------------------------------------------------------- |
| `embed-id`   | Yes      | The unique identifier for your embed: `"op://Zapier_MCP/Brand Management/mcp-embed/embed-id"` |
| `width`      | No       | Width of the embed (default: `"100%"`)                                                        |
| `height`     | No       | Height of the embed (default: `"600px"`)                                                      |
| `class-name` | No       | CSS class to apply to the iframe                                                              |

---

## Quick Account Creation

To enable Quick Account Creation, provide your user's email, first name, and last name to Zapier MCP. This will bypass Zapier sign-up for your users.

> **Note:** All three fields need to be provided in order to enable Quick Account Creation.

### Attributes

| Attribute            | Description                                                |
| -------------------- | ---------------------------------------------------------- |
| `sign-up-email`      | User's email address for quick account creation (optional) |
| `sign-up-first-name` | User's first name for quick account creation (optional)    |
| `sign-up-last-name`  | User's last name for quick account creation (optional)     |

---

## Event Reference

The `<zapier-mcp>` custom element dispatches the following DOM events:

### mcp-server-url

Fired when an MCP server URL is received. `event.detail.serverUrl` contains the URL.

### tools-changed

Fired when available tools change. Use a `tools/list` request from your MCP client to get the updated list of tools.

### close-requested

Fired when the user requests to close the embed.

---

## Integration Flow

1. **User interacts with embed**  
   When a user interacts with the Zapier MCP embed, it dispatches an `mcp-server-url` event with their unique server URL.

2. **Store the URL for the user**  
   Capture this URL and store it in your database associated with the user.

3. **Connect to Zapier MCP**  
   Use the stored URL along with your secret to connect to the user's MCP server and execute tools on their behalf.

4. **Execute tools with MCP**  
   List available tools and call them to automate workflows for your users across thousands of applications.

---

## TypeScript Implementation

### TypeScript Installation

```bash
npm install @modelcontextprotocol/sdk
```

or

```bash
pnpm add @modelcontextprotocol/sdk
```

### TypeScript Usage

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

/**
 * Connect to the Zapier MCP server
 *
 * @param serverUrl URL for the MCP server for a specific user
 * @param secret Your Zapier MCP embed secret
 * @returns MCP client
 */
async function connectToZapierMcp(serverUrl: string, secret: string) {
  const transport = new StreamableHTTPClientTransport(new URL(serverUrl), {
    requestInit: {
      headers: {
        Authorization: `Bearer ${secret}`,
      },
    },
  });

  const client = new Client({
    name: "zapier-mcp",
    version: "1.0.0",
  });

  await client.connect(transport);

  return client;
}

async function main() {
  const client = await connectToZapierMcp(serverUrl, secret);

  const tools = await client.listTools();
  console.log("Available tools:", tools);

  await client.transport?.close();
  await client.close();
}
```

---

## Python Implementation

### Python Installation

```bash
pip install mcp
```

or

```bash
uv pip install mcp
```

### Python Usage

```python
import asyncio

from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

def connect_to_zapier_mcp(server_url: str, secret: str):
    """
    Connect to the Zapier MCP server

    Args:
        server_url: URL for the MCP server for a specific user
        secret: Your Zapier MCP embed secret
    """
    return streamablehttp_client(
        server_url,
        headers={
            "Authorization": f"Bearer {secret}",
        },
    )


async def main():
    async with connect_to_zapier_mcp(server_url, secret) as (
        read_stream,
        write_stream,
        _,
    ):
        async with ClientSession(read_stream, write_stream) as session:
            await session.initialize()

            tools = await session.list_tools()
            print("Available tools:", tools)
```

---

## WordPress/WooCommerce Integration

### Overview

This integration enables WooCommerce customers to access the **Brand Management Hub Interface** (a Zapier Interface) embedded within your WordPress site. When a logged-in WooCommerce user visits the portal page, their credentials are automatically passed through to the embedded Zapier Interface iframe, enabling seamless authentication without requiring separate login.

The Zapier MCP integration runs in the background, facilitating all necessary API calls and updates that power the Brand Management Hub. This ensures customers can manage their brand assets, workflows, and automations directly from your WooCommerce site with a unified experience.

### Architecture

```plaintext
WordPress/WooCommerce Site
    ↓
Customer Login (WooCommerce Auth)
    ↓
Portal Page with Embedded Zapier Interface (iframe)
    ↓
User credentials passed through to iframe
    ↓
Zapier MCP Integration (Background)
    ↓
Handles API calls, updates, and automation for Brand Management Hub
```

### Prerequisites

- WordPress with WooCommerce installed
- Active WooCommerce customer accounts
- Access to theme files or child theme
- Zapier Interface URL for Brand Management Hub
- Zapier MCP embed credentials for background integration

---

### Implementation Steps

#### 1. Add PHP Function to Functions.php

Add this code to your theme's `functions.php` or create a custom plugin:

```php
<?php
/**
 * Get current WooCommerce user data for Zapier MCP integration
 *
 * This function retrieves logged-in user information from WordPress/WooCommerce
 * and prepares it for automatic population into the Zapier MCP embed.
 */
function get_wc_user_data_for_zapier() {
    // Check if user is logged in
    if (!is_user_logged_in()) {
        return null;
    }

    // Get current user object
    $current_user = wp_get_current_user();

    // Get WooCommerce customer object for additional data
    $customer = new WC_Customer($current_user->ID);

    // Prepare user data array
    $user_data = array(
        'email' => $current_user->user_email,
        'firstName' => $customer->get_billing_first_name() ?: $current_user->user_firstname,
        'lastName' => $customer->get_billing_last_name() ?: $current_user->user_lastname,
        'userId' => $current_user->ID,
        'username' => $current_user->user_login
    );

    // Return as JSON for JavaScript consumption
    return $user_data;
}

/**
 * Enqueue script to pass user data to JavaScript
 */
function enqueue_zapier_user_data() {
    $user_data = get_wc_user_data_for_zapier();

    if ($user_data) {
        wp_localize_script('jquery', 'zapierUserData', $user_data);
    }
}
add_action('wp_enqueue_scripts', 'enqueue_zapier_user_data');
```

#### 2. Create Portal Page Template

Create a custom page template `template-brand-management-hub.php`:

```php
<?php
/**
 * Template Name: Brand Management Hub
 * Description: Embeds Zapier Interface for Brand Management with WooCommerce user authentication
 */

get_header();

// Get current user data
$user_data = get_wc_user_data_for_zapier();

// Redirect if not logged in
if (!$user_data) {
    wp_redirect(wc_get_page_permalink('myaccount'));
    exit;
}

// Zapier Interface URL (replace with your actual interface URL)
$zapier_interface_url = 'https://interfaces.zapier.com/your-brand-management-interface';
?>

<div class="brand-management-hub">
    <div class="hub-header">
        <h1>Brand Management Hub</h1>
        <p>Welcome, <?php echo esc_html($user_data['firstName']); ?>! Manage your brand assets and workflows.</p>
    </div>

    <!-- Zapier Interface iframe container -->
    <div id="zapier-interface-container" style="width: 100%; height: 800px; border: none;">
        <!-- Interface iframe will be inserted here with user authentication -->
    </div>

    <!-- Hidden: Zapier MCP embed for background integration -->
    <div id="zapier-mcp-background" style="display: none;">
        <!-- MCP embed runs in background to handle API calls -->
    </div>
</div>

<script>
jQuery(document).ready(function($) {
    // User data is available via zapierUserData global variable
    console.log('User authenticated:', zapierUserData);
});
</script>

<?php get_footer(); ?>
```

#### 3. Embed Zapier Interface with Authentication Pass-Through

Add this JavaScript to load the Zapier Interface iframe and MCP background integration:

```javascript
<script src="https://mcp.zapier.com/embed/v1/mcp.js"></script>

<script>
jQuery(document).ready(function($) {
    // Check if user data is available
    if (typeof zapierUserData === 'undefined' || !zapierUserData) {
        console.error('User authentication data not available');
        return;
    }

    // 1. CREATE ZAPIER INTERFACE IFRAME
    // Build the interface URL with user authentication parameters
    const baseInterfaceUrl = 'https://interfaces.zapier.com/your-brand-management-interface';
    const interfaceUrl = new URL(baseInterfaceUrl);

    // Pass user credentials as URL parameters to the interface
    interfaceUrl.searchParams.append('user_email', zapierUserData.email);
    interfaceUrl.searchParams.append('user_first_name', zapierUserData.firstName);
    interfaceUrl.searchParams.append('user_last_name', zapierUserData.lastName);
    interfaceUrl.searchParams.append('user_id', zapierUserData.userId);

    // Create and insert iframe
    const iframe = document.createElement('iframe');
    iframe.src = interfaceUrl.toString();
    iframe.style.width = '100%';
    iframe.style.height = '800px';
    iframe.style.border = 'none';
    iframe.setAttribute('allow', 'clipboard-read; clipboard-write');

    document.getElementById('zapier-interface-container').appendChild(iframe);

    console.log('Brand Management Hub Interface loaded with user:', zapierUserData.email);

    // 2. INITIALIZE BACKGROUND MCP INTEGRATION
    // This runs hidden to facilitate API calls for the Brand Management Hub
    const mcpContainer = document.getElementById('zapier-mcp-background');
    const zapierMcp = document.createElement('zapier-mcp');

    // Configure MCP embed (runs in background)
    zapierMcp.setAttribute('embed-id', 'op://Zapier_MCP/Brand Management/mcp-embed/embed-id');
    zapierMcp.setAttribute('width', '1px');
    zapierMcp.setAttribute('height', '1px');

    // Auto-populate user credentials for MCP
    zapierMcp.setAttribute('sign-up-email', zapierUserData.email);
    zapierMcp.setAttribute('sign-up-first-name', zapierUserData.firstName);
    zapierMcp.setAttribute('sign-up-last-name', zapierUserData.lastName);

    mcpContainer.appendChild(zapierMcp);

    // 3. HANDLE MCP SERVER URL FOR BACKGROUND OPERATIONS
    zapierMcp.addEventListener('mcp-server-url', function(event) {
        console.log('MCP integration active for Brand Management Hub');
        const serverUrl = event.detail.serverUrl;

        // Save MCP server URL for background API operations
        $.ajax({
            url: ajaxurl,
            type: 'POST',
            data: {
                action: 'save_zapier_server_url',
                nonce: '<?php echo wp_create_nonce("save_zapier_url"); ?>',
                server_url: serverUrl,
                user_id: zapierUserData.userId
            },
            success: function(response) {
                console.log('Background MCP integration configured:', response);

                // Notify the iframe that MCP is ready
                iframe.contentWindow.postMessage({
                    type: 'mcp_ready',
                    serverUrl: serverUrl,
                    userData: zapierUserData
                }, '*');
            }
        });
    });

    // 4. LISTEN FOR MESSAGES FROM INTERFACE IFRAME
    window.addEventListener('message', function(event) {
        // Verify message origin (update with your Zapier interface domain)
        if (event.origin !== 'https://interfaces.zapier.com') {
            return;
        }

        // Handle interface requests
        if (event.data.type === 'interface_ready') {
            console.log('Brand Management Hub Interface ready');
        }

        if (event.data.type === 'update_required') {
            console.log('Interface requesting update via MCP');
            // MCP integration handles the API calls automatically
        }
    });

    // 5. HANDLE TOOLS UPDATES
    zapierMcp.addEventListener('tools-changed', function(event) {
        console.log('Brand Management Hub tools updated');
        // Notify interface of available tools update
        iframe.contentWindow.postMessage({
            type: 'tools_updated'
        }, '*');
    });
});
</script>
```

#### 4. Save MCP Server URL (Optional)

Add AJAX handler to save the MCP server URL to user meta:

```php
<?php
/**
 * AJAX handler to save Zapier MCP server URL to user meta
 */
function save_zapier_server_url_ajax() {
    // Verify nonce
    check_ajax_referer('save_zapier_url', 'nonce');

    // Get data
    $server_url = sanitize_text_field($_POST['server_url']);
    $user_id = intval($_POST['user_id']);

    // Verify user
    if ($user_id !== get_current_user_id()) {
        wp_send_json_error('Unauthorized');
        return;
    }

    // Save to user meta
    update_user_meta($user_id, 'zapier_mcp_server_url', $server_url);
    update_user_meta($user_id, 'zapier_mcp_connected_date', current_time('mysql'));

    wp_send_json_success(array(
        'message' => 'Server URL saved successfully',
        'url' => $server_url
    ));
}
add_action('wp_ajax_save_zapier_server_url', 'save_zapier_server_url_ajax');
```

---

### Security Considerations

1. **User Verification**: Always verify the logged-in user's identity before displaying their data
2. **Nonce Protection**: Use WordPress nonces for all AJAX requests
3. **Data Sanitization**: Sanitize all user input and escape output
4. **SSL/HTTPS**: Ensure your site uses HTTPS for secure data transmission
5. **Role Verification**: Optionally restrict portal access to specific WooCommerce customer roles

```php
<?php
// Example: Restrict to specific roles
function check_zapier_portal_access() {
    if (!current_user_can('customer') && !current_user_can('administrator')) {
        wp_redirect(home_url());
        exit;
    }
}
add_action('template_redirect', 'check_zapier_portal_access');
```

---

### Integration Flow Diagram

```plaintext
1. Customer logs into WooCommerce
   ↓
2. Customer navigates to Brand Management Hub page
   ↓
3. WordPress retrieves WooCommerce user credentials
   ↓
4. User data passed to JavaScript via wp_localize_script
   ↓
5. JavaScript creates Zapier Interface iframe
   ↓
6. User credentials passed as URL parameters to iframe
   ↓
7. Zapier Interface authenticates user automatically
   ↓
8. Background: Zapier MCP embed initialized (hidden)
   ↓
9. MCP handles API calls and updates for the Hub
   ↓
10. MCP server URL saved to WordPress user meta
   ↓
11. Customer uses Brand Management Hub with full functionality
    (No separate login required)
```

---

### Architecture Details

**Two-Layer Architecture:**

1. **Frontend Layer (Visible)**: Zapier Interface iframe displays the Brand Management Hub interface where users interact with their brand assets, workflows, and settings.

2. **Background Layer (Hidden)**: Zapier MCP integration runs invisibly, facilitating all API calls, data synchronization, and automation workflows that power the Hub.

**Authentication Flow:**

- WooCommerce credentials → Passed through page → Into iframe → Zapier Interface authentication
- Same credentials → Background MCP authentication → Enables API operations

This dual-layer approach ensures users see a clean, branded interface while the integration handles all technical operations seamlessly in the background.

---

### Benefits

- **Unified Experience**: Customers access the Brand Management Hub directly from your WooCommerce site
- **No Separate Login**: User credentials automatically passed through to the Zapier Interface
- **Background Integration**: MCP handles all API calls and updates without user interaction
- **Seamless Management**: Customers manage brand assets and workflows in one place
- **Consistent Authentication**: Single authentication flow for both interface and API operations
- **Improved Adoption**: Reduces friction by eliminating multiple login steps

---

### Troubleshooting

**Issue**: User data not appearing in embed

- **Solution**: Check that `wp_localize_script` is properly enqueued before the script that uses it
- **Solution**: Verify user is logged in with `is_user_logged_in()`

**Issue**: Embed not rendering

- **Solution**: Ensure the Zapier MCP script is loaded before creating the custom element
- **Solution**: Check browser console for JavaScript errors

**Issue**: MCP server URL not being saved

- **Solution**: Verify AJAX URL is correctly set (use `admin_url('admin-ajax.php')`)
- **Solution**: Check nonce verification is passing

---

## Metadata

- **Created:** 2025-12-12T06:21:17Z
- **Updated:** 2025-12-12T07:57:37Z
- **UUID:** klzyihnmuyoexudhhi3wvnlhm4
