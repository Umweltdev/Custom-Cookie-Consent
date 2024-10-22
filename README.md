# Customized GDPR Compliant Cookie Consent in React with Google Tag Manager and Analytics Integration
## 1. Introduction
Implementing a GDPR-compliant cookie consent mechanism in your React project is a great way to ensure transparency and give users control over their data while integrating tools like Google Tag Manager (GTM) and Google Analytics (GA). Below, I'll walk you through a detailed approach that includes setting up the cookie consent banner, connecting it to Google Tag Manager and Google Analytics, and managing the switches for different types of cookies (Personalization, Marketing, Analytics, and Optimization).

## 2. Understanding GDPR and Cookie Consent
GDPR consent is the explicit permission given by individuals for their personal data to be processed. It must be
Under the GDPR, websites must:
 * Obtain clear and explicit consent from users before collecting any personal data (including cookies).
* Allow users to manage their cookie preferences (accept or reject different categories).
* Provide users the ability to withdraw consent at any time.

## 3. Define Requirements for GDPR Compliance
* Explicit User Consent: Consent must be obtained before placing any cookies.
* Granular Control: Users should be able to toggle categories of cookies (Personalization, Marketing, Analytics, Optimization).
* User Consent Storage: Consent decisions must be stored in localStorage or cookies.
* Revocation: Users should be able to revoke consent at any time.
* Blocking Google Analytics and Tag Manager: Ensure GA and GTM only run when consent is granted.
  
## 4. Setting up the React Cookie Consent Banner
You can use the react-cookie-consent package like:
```
npm install react-cookie-consent
```
However for this we will use customized cookies,this will help us to leverage on defining preferences for the user:
Below is a customized cookies banner with four preferences set,user can eithter accept or reject all, or set preferences and save settings. 
```
import * as React from "react";
import { useState, useEffect } from "react";
import { styled } from "@mui/material/styles";
import ArrowForwardIosSharpIcon from "@mui/icons-material/ArrowForwardIosSharp";
import MuiAccordion from "@mui/material/Accordion";
import MuiAccordionSummary from "@mui/material/AccordionSummary";
import MuiAccordionDetails from "@mui/material/AccordionDetails";
import Typography from "@mui/material/Typography";
import { Box, Button, Card, IconButton, Modal, Switch } from "@mui/material";
import { Link, useNavigate } from "react-router-dom";
import { Add, Remove } from "@mui/icons-material";

const modalStyle = {
  position: "absolute",
  top: "50%",
  left: "50%",
  transform: "translate(-50%, -50%)",
  width: 400,
  bgcolor: "background.paper",
  boxShadow: 24,
  borderRadius: 3,
  p: 4,
};

const consentData = [
  {
    id: "Personalization",
    title: "Tailor Advertising and Content Based on Limited Data",
    summary:
      "Advertising and content can be personalized based on limited data such as device type, location, and previous interactions. This ensures relevant ads and content without overwhelming the user.",
    details: [
      "Advertising: Ads tailored based on general geographic location, device type, and past interactions to avoid repetition.",
      "Service-Specific Advertising: Limits ad frequency and focuses on services aligned with your interests.",
      "Content Customization: Content adjusted to your device and location, ensuring it’s relevant and optimized.",
    ],
  },
  {
    id: "Analytics",
    title: "Understand Audiences and Measure Ad Performance",
    summary:
      "By analyzing user data, we generate insights and measure the effectiveness of advertising, helping refine marketing strategies and optimize audience engagement.",
    details: [
      "Audience Insights: Analyze cross-platform user data to optimize content and marketing strategies.",
      "Ad Performance: Track ad engagement to understand which placements lead to user interaction and optimize future campaigns.",
    ],
  },
  {
    id: "Optimization",
    title: "Optimize Device Interaction and Service Delivery",
    summary:
      "Device characteristics are scanned to optimize content delivery, while user data helps improve and develop new services.",
    details: [
      "Device Optimization: Content is tailored to the device’s specifications, providing a better user experience.",
      "Service Development: Analyze user interactions to improve services and develop new features.",
    ],
  },
  {
    id: "Enhancement",
    title: "Enhance Services Based on User Feedback and Functional Needs",
    summary:
      "We use feedback and functional cookies to improve user experience, focusing on platform functionality and evolving services based on user input.",
    details: [
      "Feedback: User suggestions influence the development of new features and integrations.",
      "Functional Cookies: Remember preferences like login information and search settings to improve user experience.",
    ],
  },
];

const BoldSwitch = styled(Switch)(({ theme }) => ({
  width: 60,
  height: 32,
  padding: 0,
  "& .MuiSwitch-switchBase": {
    padding: 2,
    "&.Mui-checked": {
      transform: "translateX(28px)",
      color: "#fff",
      "& + .MuiSwitch-track": {
        backgroundColor: "primary.main",
        opacity: 1,
      },
    },
  },
  "& .MuiSwitch-thumb": {
    width: 28,
    height: 28,
  },
  "& .MuiSwitch-track": {
    borderRadius: 34 / 2,
    backgroundColor: "#E9E9EA",
    opacity: 1,
  },
}));

const SmallSwitch = styled(Switch)(({ theme }) => ({
  width: 40, // Reduced from 60
  height: 20, // Reduced from 32
  padding: 0,
  "& .MuiSwitch-switchBase": {
    padding: 2,
    "&.Mui-checked": {
      transform: "translateX(18px)", // Adjusted to fit smaller size
      color: "#fff",
      "& + .MuiSwitch-track": {
        backgroundColor: theme.palette.primary.main,
        opacity: 1,
      },
    },
  },
  "& .MuiSwitch-thumb": {
    width: 16, // Reduced from 28
    height: 16, // Reduced from 28
  },
  "& .MuiSwitch-track": {
    borderRadius: 20 / 2, // Adjusted to match smaller height
    backgroundColor: "#E9E9EA",
    opacity: 1,
  },
}));

function ConsentBar() {
  const [open, setOpen] = useState(false);
  const [selectedConsent, setSelectedConsent] = useState(null);
  const [consentChoices, setConsentChoices] = useState({
    Personalization: false,
    Analytics: false,
    Optimization: false,
    Enhancement: false,
  });
  const [allAccepted, setAllAccepted] = useState(false);
  const [consentVisible, setConsentVisible] = useState(true);

  useEffect(() => {
    const storedConsent = JSON.parse(localStorage.getItem("user_consent"));
    if (storedConsent) {
      setConsentChoices(storedConsent);
    }
  }, []);

  const handleOpen = (data) => {
    setSelectedConsent(data);
    setOpen(true);
  };

  const handleClose = () => setOpen(false);

  const handleSwitchToggle = (id) => (event) => {
    const updatedChoices = { ...consentChoices, [id]: event.target.checked };
    setConsentChoices(updatedChoices);
    localStorage.setItem("user_consent", JSON.stringify(updatedChoices));

    // GTM event for individual consent change
    window.dataLayer?.push({
      event: "cookie_consent_updated",
      consentType: id,
      consentValue: event.target.checked ? "granted" : "rejected",
    });
  };

  const handleAcceptAll = () => {
    const allTrueChoices = Object.keys(consentChoices).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setConsentChoices(allTrueChoices);
    setAllAccepted(true);
    localStorage.setItem("user_consent", JSON.stringify(allTrueChoices));

    window.dataLayer?.push({
      event: "cookie_consent_given",
      consent: "all_granted",
    });

    setConsentVisible(false);
  };

  const handleRejectAll = () => {
    const allFalseChoices = Object.keys(consentChoices).reduce((acc, key) => {
      acc[key] = false;
      return acc;
    }, {});
    setConsentChoices(allFalseChoices);
    setAllAccepted(false);
    localStorage.setItem("user_consent", JSON.stringify(allFalseChoices));

    window.dataLayer?.push({
      event: "cookie_consent_rejected",
      consent: "all_rejected",
    });

    setConsentVisible(false);
  };

  const handleSaveSettings = () => {
    localStorage.setItem("user_consent", JSON.stringify(consentChoices));

    // GTM event for saving settings
    window.dataLayer?.push({
      event: "cookie_consent_saved",
      consent: consentChoices,
    });

    setConsentVisible(false);
  };

  if (!consentVisible) return null;

  return (
    <Box sx={{ pt: "20px 0", width: "100%" }}>
      <Box
        sx={{
          position: "fixed",
          bottom: 0,
          color: "black",
          backgroundColor: "#fff",
          width: "50vw",
          // pb: 3,
          p: 4,
          borderRadius: 3,
          mb: 4,
          "@media (max-width: 600px)": {
            width: "95vw",
            p: 3,
          },
        }}
      >
        <Typography
          variant="h5"
          mb={3}
          sx={{
            "@media (max-width: 600px)": {
              display: "none",
            },
          }}
        >
          Welcome to Prevail Consent Management
        </Typography>
        <Typography
          variant="subtitle2"
          sx={{
            "@media (max-width: 600px)": {
              display: "none",
            },
          }}
        >
          At Prevail, your privacy and control over your data are our top
          priorities. We believe in transparency, which is why we provide clear,
          detailed descriptions of how each cookie on our platform functions,
          tools, and marketing efforts and how they enhance your experience.
        </Typography>
        <Box sx={{ display: "flex", gap: 1, mt: 1 }}>
          <Button
            onClick={handleRejectAll}
            variant="outlined"
            sx={{ borderRadius: 10, textTransform: "capitalize" }}
          >
            Reject All
          </Button>
          <Button
            onClick={handleAcceptAll}
            variant="contained"
            sx={{ borderRadius: 10, textTransform: "capitalize", boxShadow: 0 }}
          >
            Accept All
          </Button>
        </Box>

        <Typography variant="subtitle2" fontWeight="600" sx={{ mt: 3 }}>
          Cookies will help us serve you better in these ways:
        </Typography>

        <Card
          sx={{
            display: "flex",
            justifyContent: "space-between",
            alignItems: "center",
            gap: 1,
            px: 10,
            mt: 1,
            p: 3,
            bgcolor: "#F5F3FB",
            boxShadow: 0,
            "@media (max-width: 600px)": {
              p: 2,
              flexDirection: "column",
              px: 2,
              gap: 0.2,
            },
          }}
        >
          {consentData.map((data, index) => (
            <Box
              key={data.id}
              sx={{
                display: "flex",
                flexDirection: "column",
                alignItems: "center",
                "@media (max-width: 600px)": {
                  flexDirection: "row",
                  justifyContent: "space-between",
                  width: "100%",
                },
              }}
            >
              <Box
                sx={{
                  display: "flex",
                  flexDirection: "column",
                  alignItems: "center",
                  "@media (max-width: 600px)": {
                    display: "block",
                  },
                }}
              >
                <Typography variant="h6">{data.id}</Typography>
                <Typography
                  variant="subtitle2"
                  mb={2}
                  color="primary"
                  sx={{ cursor: "pointer" }}
                  onClick={() => handleOpen(data)}
                >
                  Details
                </Typography>
              </Box>
              <Box
                sx={{
                  "@media (max-width: 600px)": {
                    display: "none",
                  },
                }}
              >
                <BoldSwitch
                  checked={consentChoices[data.id]}
                  onChange={handleSwitchToggle(data.id)}
                />
              </Box>
              <Box
                sx={{
                  display: "none",
                  "@media (max-width: 600px)": {
                    display: "block",
                  },
                }}
              >
                <SmallSwitch
                  checked={consentChoices[data.id]}
                  onChange={handleSwitchToggle(data.id)}
                />
              </Box>
            </Box>
          ))}
        </Card>
        <Button
          onClick={handleSaveSettings}
          variant="contained"
          sx={{
            borderRadius: 10,
            textTransform: "capitalize",
            mt: 2,
            display: "flex",
            justifyContent: "flex-end",
            textAlign: "flex-end",
            boxShadow: 0,
          }}
        >
          Save Settings
        </Button>
      </Box>

      <Modal open={open} onClose={handleClose}>
        <Box sx={modalStyle}>
          {selectedConsent && (
            <>
              <Typography id="modal-title" variant="h6">
                {selectedConsent.title}
              </Typography>
              <Typography id="modal-description" sx={{ mt: 2 }}>
                {selectedConsent.summary}
              </Typography>
              <Typography id="modal-details" sx={{ mt: 2 }}>
                {selectedConsent.details.map((detail, index) => (
                  <li key={index}>{detail}</li>
                ))}
              </Typography>
            </>
          )}
        </Box>
      </Modal>
    </Box>
  );
}

export default ConsentBar;

```

The above code is a react component and based on material ui design, and this is a break down:
* The javascript object holds in details all the consent user has to subscribe to
* there are 3 important buttons, the accept all sets the user to accept all consent, just as the opposite rejects all consent
* the Save Settings saves only the ones the user has selected to save.'

## 5. Google Tag Manager Setup
To integrate Google Tag Manager (GTM) with cookie consent, you need to follow these steps:
### a. Create a GTM Container
* Go to Google Tag Manager.
* Create a new account and container (if not done yet).
* Add the GTM script tag to your React project:

  In public/index.html, add the following in the <head> section
```
<!-- Google Tag Manager Example Code -->
<script>
(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-XXXXXX'); // Replace with your GTM ID
</script>
<!-- End Google Tag Manager -->

```

Then, just below the opening <body> tag:
```
<!-- Google Tag Manager (noscript) -->
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXX"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<!-- End Google Tag Manager (noscript) -->

```


### b. Triggering GTM with Cookie Consent
After you have added Google Tag Manager (GTM) to your React project, the goal is to configure GTM to fire only after consent is given by the user. This will allow you to load Google Analytics or other tracking scripts based on consent. Here’s how to do it:

### a. Create a 1st Party Cookie Variable in GTM
* Log in to your GTM account and select the appropriate container (created during initial GTM setup).
* On the left sidebar, go to Variables and click on the New button to create a new variable.
* In the Variable Configuration, choose 1st Party Cookie from the list of available variable types.

#### Configure the variable:
* Variable Name: cookieConsent
* Cookie Name: CookieConsent (or the same name as the cookie you set using react-cookie-consent library in your React app).
* Click Save.

This variable will be used to check the consent status stored in cookies. If cookieConsent cookie contains a value that indicates the user has accepted cookies, GTM will proceed to fire relevant tags.

### b. Create a Custom Event Trigger Based on Consent
Next, we need to create a trigger that will fire only if the consent event (like cookie_consent_given) has been received.
* Go to the Triggers section in GTM and click on New.
* Set the Trigger Configuration as follows:

#### Trigger Type: Custom Event
* Event Name: cookie_consent_given (this should match the event name you pushed in your React code when consent is given).
* Set this trigger to fire on All Custom Events.
* Click Save.

Now, this trigger will listen for the cookie_consent_given event from your React app and will allow you to fire GTM tags only when consent is provided.

## 6. Sending Consent Events to Google Analytics
To ensure that your Google Analytics setup is also compliant with GDPR and triggered only when users give consent, follow these steps.

### a. Create Google Analytics Tags in GTM
Create a Google Analytics Pageview Tag:
* Go to the Tags section and click on New.
* Tag Type: Select Google Analytics: Universal Analytics (for GA3) or Google Analytics: GA4 Configuration (for GA4).
* Track Type: Select Page View.
* In Triggering, select the trigger you created earlier (cookie_consent_given).
* Fire GA Events on Consent:

#### Create another tag for GA Event that fires based on user interactions after consent:
* Tag Type: Google Analytics: Universal Analytics or GA4 Event.
* For Universal Analytics:
* Category: Consent
* Action: Given
* Label: GDPR
* 
#### For GA4:
* Configure the event parameters (e.g., consent_status).
* Set the trigger to cookie_consent_given (same trigger as above).
  
## b. Test Your GA Tags
Preview the GTM container (click the "Preview" button in GTM).
Open your site and interact with the cookie banner.
Accept cookies, and verify that the GA tag fires only after consent is given.
Check the Tag Assistant in GTM to see the sequence of tag firing and ensure no tracking occurs before consent.

## 7. Conclusion
Now you have a GDPR-compliant cookie consent system integrated with Google Tag Manager and Google Analytics, and you've set it up to handle multiple cookie categories like Analytics and Marketing.

This setup ensures that tracking or marketing scripts are only loaded after the user gives explicit consent, fulfilling the GDPR requirements.
