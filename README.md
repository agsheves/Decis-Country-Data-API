# Decis Intelligence Country Info API
The code repository and API docs for Decis users

**DRAFT - these documents currently refer to the test version of the endpoint.**

# Overview of the API

The Decis research API allows authorized users to access the Decis research database to access up-to-date country news, intelligence, and analysis for use in their own products. Access to the API endpoint is via **HTTPS GET request** initiated by the user and requires valid credentials for access. 

The request is made on a per-country basis or configured based on the client's credentials. The API call output is in JSON format, explained in detail below.

# Authentication

Users require a valid user ID and key to access the API. These credentials are unique to the named user and are not to be shared. Teams can have multiple users, but each must use their own set of credentials. Each access attempt is logged, tracking the supplied username and the originating IP address. Users who appear to be sharing their credentials may lose access to the API.

## Request access

[Email us to request access](mailto:support@decis.ai) include ```API Key Request``` in the subject

**Ensure that you include details of your organization and potential use case**

# Endpoint

The endpoint address is ```https://countryassessments.anvil.app/_/api/countrydata-test-closed```

The endpoint can be accessed as follows.

## Python

```
import requests

def get_country_data(email, api_key):
    url = 'https://countryassessments.anvil.app/_/api/countrydata-test-closed'
    response = requests.get(url, auth=(email, api_key))
    
    if response.status_code == 200:
        return response.json()
    else:
        return f"Error: {response.status_code} - {response.text}"

```

## CURL

```
$ curl -u me@example.com:my_api_key https://countryassessments.anvil.app/_/api/countrydata-test-closed

(Replace 'me@example.com' with your registered username and 'my_api_key' with your personal Decis API key
```


# Response Structure
Results from the API are presented in JSON with the following fields. (Items marked with an * are provided in the test API.)

| Field                                | Format                | Description |
| ------------------------------------ | --------------------- | ----------- |
|`country_ref_code`                    |    text                | The ISO-3 reference for the country sourced from the [CountryInfo python library](https://pypi.org/project/countryinfo/target=blank) (link leaves this page). This is the best key to use for accurate and consistent country identification.      |
| `country_name` *                      | text                  | The ISO standard name for the country.  |
|`flag` *                               | text (unicode emoji) | Emoji version of the country's flag. |
| `country_summary`                    | text                  | An aggregated summary of the country from multiple sources, presented as a single paragraph of text. |
| `baseline_assessment` *              | text                  | An assessment of the baseline stability rating using the Decis ratings (see below for more). |
| `custom_rating`                      | text                  | An assessment of the baseline stability rating using the user's custom ratings (see below for more). This is only returned where a custom rating system has been established, otherwise this is skipped. |
| `latest72hrAssessmentDate`           | datetime string       | The date and time of the last 72-hour assessment, returned as a full datetime string including timezone in ISO 8601 format. |
| `latest_directional_assessment` *    | text                  | An assessment of the comparative stability for the country at the moment compared to the baseline  ratings (see below for more). |
| `short_term_assessment_period`       | text                  | The 72-hour assessment span in text format (e.g., "This assessment covers the period 12-15 February 2024"). |
| `last_midterm_assessment_date`       | date string           | The date of the last mid-term (30-90 day) assessment as a simple date string in ISO 8601 format (e.g., "2024-02-15"). |
| `latest_news`                        | text                  | A summary of the latest events in text form without links or sources. |
| `mid_term_events`                    | text                  | A summary of upcoming events that were used to conduct the mid-term analysis. |
| `mid_term_assessment`                | text                  | An assessment of the assessed stability for the upcoming 30-90 day period using the Decis ratings (see below for more). |

# Ratings and Customization

## Decis Standard Ratings

Decis stability ratings are based on a five-step structure based on an assessment of a location using 350+ stability pairs.

| Rating term | Description|
| ------------ | ---------- |
| Very Stable | Indicates a country where there are very high levels of rule of law, income, education, social harmony and environmental stability  |
| Stable | Indicates a country where there are high levels of rule of law, income, education, social harmony and environmental stability but there are some areas of tension or limitations |
| Weak | Indicates a country where there are significant areas of shortfall or tension and societal cohesion is under strain | 
| Unstable | Many of the necessary foundations of a functioning society have or are breaking down. Often accompanied by areas or regions where civil unrest or conflict are breaking out.  | 
| Very Unstable | Widespread breakdown of societal support structures and cohesion. Often accompanied by widespread civil unrest or conflict.| 


## Custom Rating Terms

Users can request custom ratings that align with this system. These custom results are linked to a particular user or team and are only included in responses to requests submitted with the appropriate credentials. Otherwise, these are skipped, and only the standard Decis ratings are returned.
Note that custom terms are a terminology change only: the underlying Decis methodology remains the same. Therefore, custom ratings are unavailable when terminologies cannot be aligned.

## Questions

Please [email support](mailto:support@decis.ai) with any questions

## What's under the hood?

<a href="https://groq.com" target="_blank" rel="noopener noreferrer">
  <img
    src="https://groq.com/wp-content/uploads/2024/03/PBG-mark1-color.svg"
    alt="Powered by Groq for fast inference."
    style="width: 25%"
  />
</a>




