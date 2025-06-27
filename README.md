# 🚀 Leveraging AI in Python to Automatically Identify Businesses by Address

In today’s digital era, **Artificial Intelligence (AI)** plays a vital role in automating and enhancing business workflows.  
I recently developed a **Python-based tool** that integrates **OpenAI’s** model to extract public business information — using nothing but a physical address.

---

## 🔍 What Does This Tool Do?

- 💬 **Asks** an AI model what company (if any) is located at a given address  
- 🧾 **Returns** the business name, contact details, website, and a short description (if available)  
- ⏱️ **Saves time** by automating business lookup and verification  

---

## 💼 Potential Use Cases for Businesses

- ✅ **Verifying potential partners or vendors**  
- 📊 **Market research and competitor analysis**  
- 🗂️ **Automating the creation of customer/supplier databases**  
- 📈 **Enhancing sales and marketing intelligence**  

This project shows how AI can be seamlessly embedded into day-to-day operations, enabling **smarter decisions**, faster.
 
 

---

## 🧠 Python Code Example

```python
import requests
from math import radians, sin, cos, sqrt, atan2
import openai

# API Keys
GOOGLE_API_KEY = "YOUR_GOOGLE_API_KEY"
OPENAI_API_KEY = "YOUR_OPENAI_API_KEY"
client = openai.OpenAI(api_key=OPENAI_API_KEY)

# Get coordinates from address using Nominatim
def geocode(address):
    url = "https://nominatim.openstreetmap.org/search"
    params = {"q": address, "format": "json", "limit": 1}
    headers = {"User-Agent": "BusinessLookupBot/1.0"}
    r = requests.get(url, params=params, headers=headers)
    data = r.json()
    if data:
        lat, lon = data[0]["lat"], data[0]["lon"]
        print(f" Coordinates of the address: {lat}, {lon}")
        return lat, lon
    return None, None

# Find nearby businesses using Google Places API
def places_nearby(lat, lon, radius=150):
    url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
    params = {
        "location": f"{lat},{lon}",
        "radius": radius,
        "key": GOOGLE_API_KEY,
    }
    r = requests.get(url, params=params)
    return r.json().get("results", [])

# Get detailed info from Google Place Details API
def get_place_details(place_id):
    url = "https://maps.googleapis.com/maps/api/place/details/json"
    params = {
        "place_id": place_id,
        "fields": "name,formatted_phone_number,website,vicinity",
        "key": GOOGLE_API_KEY,
    }
    r = requests.get(url, params=params)
    return r.json().get("result", {})

# Generate a short summary using OpenAI
def summarize_business_info(info_json):
    prompt = f"""
Based on the following data, write a short business description (max 20 words):

{info_json}
"""
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}],
        temperature=0
    )
    return response.choices[0].message.content.strip()

def distance_km(lat1, lon1, lat2, lon2):
    R = 6371
    dlat = radians(float(lat2) - float(lat1))
    dlon = radians(float(lon2) - float(lon1))
    a = sin(dlat / 2) ** 2 + cos(radians(float(lat1))) * cos(radians(float(lat2))) * sin(dlon / 2) ** 2
    c = 2 * atan2(sqrt(a), sqrt(1 - a))
    return R * c

def find_business_by_address(address):
    lat, lon = geocode(address)
    if not lat:
        return " Address not found."

    places = places_nearby(lat, lon)
    if not places:
        return " No businesses found near this address."

    closest = min(places, key=lambda p: distance_km(
        lat, lon,
        p["geometry"]["location"]["lat"],
        p["geometry"]["location"]["lng"]
    ))

    details = get_place_details(closest["place_id"])
    summary = summarize_business_info(details)

    return f"""
 Business Name: {details.get('name', 'Not available')}
 Phone: {details.get('formatted_phone_number', 'Not available')}
 Website: {details.get('website', 'Not available')}
 Address: {details.get('vicinity', 'Not available')}
 Summary: {summary}
"""
if __name__ == "__main__":
    address_input = input("Enter the address to look up: ").replace(";", ",").strip()
    result = find_business_by_address(address_input)
    print("\n🧾 Business Lookup Result:\n")
    print(result)

