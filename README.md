# Temp Mail Detector Python API

This repository contains the required code for you to make an API request to the [Temp Mail Detector](https://tempmaildetector.com) service using Python.

Temporary email addresses can cause issues for services which provide a freemium model or which offer a trial. While we understand that temporary emails are great at preserving privacy, there is a need to control where and when they can be used.

Below you will find an example implementation and JSON response of this library:

### Example response
```json
{
  "domain": "apn7.com",
  "score": 100,
  "meta": {
    "block_list": true,
    "domain_age": 2,
    "website_resolves": false,
    "accepts_all_addresses": false,
    "valid_email_security": true
  }
}
```

### Example usage
```python
import json
import requests
from dataclasses import dataclass

API_URL = "https://api.tempmaildetector.com/check"
CONTENT_TYPE = "application/json"

@dataclass
class Meta:
    block_list: bool
    domain_age: int
    website_resolves: bool
    accepts_all_addresses: bool
    valid_email_security: bool

@dataclass
class DomainCheckResponse:
    domain: str
    score: int
    meta: Meta

class Client:
    def __init__(self, api_key: str):
        self.api_key = api_key

    def check_domain(self, domain: str) -> DomainCheckResponse:
        request_body = json.dumps({"domain": domain})
        headers = {
            "Content-Type": CONTENT_TYPE,
            "Authorization": self.api_key
        }

        response = requests.post(API_URL, data=request_body, headers=headers)
        
        if response.status_code != 200:
            raise Exception(f"Received non-200 response: {response.text}")

        response_data = response.json()
        meta = Meta(
            block_list=response_data["meta"]["block_list"],
            domain_age=response_data["meta"]["domain_age"],
            website_resolves=response_data["meta"]["website_resolves"],
            accepts_all_addresses=response_data["meta"]["accepts_all_addresses"],
            valid_email_security=response_data["meta"]["valid_email_security"],
        )
        return DomainCheckResponse(
            domain=response_data["domain"],
            score=response_data["score"],
            meta=meta
        )

# Example usage:
if __name__ == "__main__":
    api_key = "YOUR_API_KEY"
    client = Client(api_key)

    domain = "devncie.com"
    try:
        response = client.check_domain(domain)
        print(f"Domain: {response.domain}\nScore: {response.score}\nMeta: {response.meta}")
    except Exception as e:
        print(f"Error checking domain: {e}")
```