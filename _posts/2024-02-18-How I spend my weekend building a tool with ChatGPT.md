---
layout: post
date: 2024-02-10
title: How I spend my weekend building a tool with ChatGPT
giscus_comments: true
related_posts: false
tags: llm productivity
---
### Summary
I've been using ChatGPT for things I know the answer to and it's working fairly well. I've been looking for ways to go further and start using it to explore things I do not know as well. This weekend I had a good excuse to do that. 

I wanted to transfer my saved places from a google account to another account. Surprisingly Google doesn't give you that option. That was actually the perfect excuse to use ChatGPT to do so build a tool over the weekend for me. 

My goal here is not only to build a new tool, but to also test ChatGPT's abilities. I am interested to understand how could ChatGPT function as a general purpose *learning copilot*. I am looking for things like its usefulness in breaking a problem down, implementing each step, explaining along the way its thinking, zooming in as needed to explain a new concept, zooming out to review the plan so far and so on. To this end I ask it a mix a questions of things I know about and things I know less about. I found that ChatGPT provides value in both cases.

**Conclusions**
ChatGPT can break down and trouble shoot fairly simple problems and suggest reasonable solutions. It's OK at zooming in to troubleshoot a specific issue, and zooming back out to continue the larger plan, but it helps to remind it. In that sense it's certainly a useful learning copilot and I imagine it can be significantly  more helpful as we finetune it to specific use cases. 

Another interesting insight for me was that by asking it questions for which I know the answer to, I usually ended up learning something new. By prompting the LLM to suggest more than one solutions to a problem, we can expand our creativity. I believe this use case can be a really helpful feature of LLMs. LLMs operate quite differently from humans. One of their unique strengths is the ability to bring immense volumes of data within the context of a conversation very quickly.  Humans can use this ability to expand the set of approaches to a specific problem, hence allowing us to not just to arrive faster to the solution, but potentially arrive to a better solution or more than one solutions. 


### Development Log
In the following section I describe my experience "working with" ChatGPT to build a new tool. The purpose of the tool is to help transfer a list of saved places from one Google account to another. 

I am an OK programmer, but not a full stack developer. I figured I'll need some help with this. I'll make a point to ask questions at each step, including to things I know the answer to. I opened ChatGPT and explained the task. It said that the  recommended  way was to manually enter each item in. *But*, it also gave an alternative approach which would require "programming knowledge and familiarity with Google's APIs.". Let's do that, I said. 

Great, it said here is an approach. 

Here is an example list of saved places 
```json
{

"type": "FeatureCollection",

"features": [

{

"geometry": {

"coordinates": [

2.3567354,

48.8517232

],

"type": "Point"

},

"properties": {

"date": "2022-07-09T17:15:34Z",

"google_maps_url": "http://maps.google.com/?cid=3697878718076448074",

"location": {

"address": "31 rue saint louis en l'ile, 46 Rue Saint-Louis en l'Île, 75004 Paris, France",

"country_code": "FR",

"name": "Berthillon"

}

},

"type": "Feature"

},

{

"geometry": {

"coordinates": [

28.9777673,

41.0583419

],

"type": "Point"

},

"properties": {

"date": "2022-02-19T20:03:57Z",

"google_maps_url": "http://maps.google.com/?cid=11086473770627030104",

"location": {

"address": "Cumhuriyet Hacıahmet Silahşör Cad, Yeniyol Sk. No:2, 34440 Şişli/İstanbul, Türkiye",

"country_code": "TR",

"name": "TURK FATIH TUTAK"

}

},

"type": "Feature"

}

]

}
```


After downloading the json file I wanted to try out the code that chatgpt provided. To avoid having to setup local environments and to allow other people to run the code, I wanted to run it in Google colab if possible. I asked ChatGPT if it agreed and it said sure and showed me how to start a notebook and upload my "Saved Places.json" file to the directory. 

I then asked how to start coding and offered this
```python
import json

# Load the JSON file
with open('Saved Places.json', 'r') as file:
    data = json.load(file)

# Assuming your JSON has a list of places, adjust the keys as necessary
saved_places = []  # This will hold tuples of (name, latitude, longitude)
for item in data:  # Adjust this loop based on the structure of your JSON file
    try:
        name = item['name']  # Adjust this key based on your JSON structure
        latitude = item['geometry']['location']['lat']  # Adjust path as necessary
        longitude = item['geometry']['location']['lng']  # Adjust path as necessary
        saved_places.append((name, latitude, longitude))
    except KeyError:
        print("Key not found in item:", item)
        continue

# Print out the first few items to verify
for place in saved_places[:5]:  # Print first 5 places to check
    print(place)
```

I ran that and got an error 
```---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-7-8cbb47bb541b> in <cell line: 9>()
      9 for item in data:  # Adjust this loop based on the structure of your JSON file
     10     try:
---> 11         name = item['name']  # Adjust this key based on your JSON structure
     12         latitude = item['geometry']['location']['lat']  # Adjust path as necessary
     13         longitude = item['geometry']['location']['lng']  # Adjust path as necessary

TypeError: string indices must be integers
```

hm... I can fix this but before I try that I notice the button just below "explain  error". Indulging a digresion into Google colab's debugging abilities I clicked it and (after agreeing to all terms and conditions) I got a new window explaining the errors and suggesting better code. Huh, not bad!

It says try this instead
```python
saved_places = []

for item in data['features']:  # Adjust this loop based on the structure of your JSON file
    try:
        name = item['properties']['name']  # Adjust this key based on your JSON structure
        latitude = item['geometry']['coordinates'][1]  # Adjust path as necessary
        longitude = item['geometry']['coordinates'][0]  # Adjust path as necessary
        saved_places.append((name, latitude, longitude))
    except KeyError:
        print("Key not found in item:", item)
        continue
```


This now outputs
```json
Key not found in item: {'geometry': {'coordinates': [2.3567354, 48.8517232], 'type': 'Point'}, 'properties': {'date': '2022-07-09T17:15:34Z', 'google_maps_url': 'http://maps.google.com/?cid=3697878718076448074', 'location': {'address': "31 rue saint louis en l'ile, 46 Rue Saint-Louis en l'Île, 75004 Paris, France", 'country_code': 'FR', 'name': 'Berthillon'}}, 'type': 'Feature'}

Key not found in item: {'geometry': {'coordinates': [28.9777673, 41.0583419], 'type': 'Point'}, 'properties': {'date': '2022-02-19T20:03:57Z', 'google_maps_url': 'http://maps.google.com/?cid=11086473770627030104', 'location': {'address': 'Cumhuriyet Hacıahmet Silahşör Cad, Yeniyol Sk. No:2, 34440 Şişli/İstanbul, Türkiye', 'country_code': 'TR', 'name': 'TURK FATIH TUTAK'}}, 'type': 'Feature'}
```


Ok let's fix this and move on. Back to asking ChatGPT:

```
User: Let's explore the automated way of adding places, and let's use Google IDs too. Here is what we have so far: 
- a file "saved places.json" 
- a google colab notebook running with access to the file 
- the working script which can read the file and extract info from the json, including name and location
```

It returns the following code to get the Google IDs

```python
import requests
import json

# Function to find a place ID given its name and location
def find_place_id(api_key, name, latitude, longitude):
    endpoint_url = "https://maps.googleapis.com/maps/api/place/findplacefromtext/json"
    params = {
        'input': name,
        'inputtype': 'textquery',
        'fields': 'place_id',
        'locationbias': f'point:{latitude},{longitude}',
        'key': api_key
    }
    response = requests.get(endpoint_url, params=params)
    result = response.json()
    if result['candidates']:
        return result['candidates'][0]['place_id']
    else:
        return None

# Your Google API key
api_key = 'YOUR_API_KEY_HERE'

# Enhanced loop to include Place IDs
for item in data['features']:
    try:
        name = item['properties']['location']['name']
        latitude = item['geometry']['coordinates'][1]
        longitude = item['geometry']['coordinates'][0]
        # Find Place ID
        place_id = find_place_id(api_key, name, latitude, longitude)
        saved_places.append((name, latitude, longitude, place_id))
    except KeyError as e:
        print(f"Key not found {e} in item:", item)
        continue

# Print to verify
for place in saved_places[:5]:
    print(place)

```

Great - but I don't have a Google API key. I ask how to get it and it guides me through the process. But before we move on, I ask one more thing (I would have used a `.env` file but I wanted to ask the question to see what it said): 
```
User: OK show me how to secure my API key, so that I can share the code in a safe way
```

It gives some good suggestions, including prompting the user for the API key in the code with two lines of code: 
```python
from getpass import getpass
api_key = getpass('Enter your API key:')
```
Interesting - hadn't thought of that. I'll use the suggestion, I like it more than having to load an environment variable. 

On to the next step. Automating uploading saved places from a json to a Google account. ChatGPT suggested a manual approach which I am trying to avoid but it's certainly reasonable. This shows some ability to provide the most reasonable answer first. I asked it for more than one solutions and gave me four solutions: 
1. Manual Addition with Enhanced Support
2. Leverage Google My Maps
3. Use Google Maps API for Custom Applications
4. Share and Collaborate

All four are reasonable and I didn't think of all four originally. That's a helpful context to have as I am trying to solve this problem. Let's pick number 3 and move on. We will try a browser automator tool and specifically we will use Selenium, as ChatGPT suggested. Let's explore this idea. It provided instructions for doing that including the following snippet.
```
### Important Considerations

- **Rate Limiting and Blocking:** Automated interactions can trigger rate limiting or blocking by Google, especially for login and frequent actions. Use delays (`time.sleep()`) to mimic human behavior and reduce the risk of being blocked.
- **Google's Terms of Service:** Ensure your automation respects Google's terms of service. Automating certain actions may violate these terms.
- **Maintainability:** Google Maps' UI can change, which may require you to update your script to match the new structure.

Automating interactions with web services like Google Maps should be approached with caution, respecting usage policies and ethical considerations. For personal use, automation can save time, but ensure your actions don't impact the service's operation for others.
```
That's helpful information which as we will see shortly are an issue. ChatGPT's ability to provide these potential limitations of this approach up front, while exploring this approach, is certainly helpful context while we are problem solving. 

In fact, after going through with this solution I ran into two issues: 
 - running selenium in a google colab was not possible 
 - selenium cannot work at all for anything that requires google authentication
ChatGPT was able to help me explore this solution with helpful snippets of code and with trouble shooting advice. And it was able to tell me about these limitations up front when I was devising a plan. 

In the end I ran out of time. By the end of my working session I was not able to create a tool that can transfer a list of saved places from one google account to another. I got stuck at combining browser automation with the need to log in to a google account. I expect this is intentionally hard for security purposes. 

Nevertheless I learned a lot in the process, including the fact ChatGPT can help us expand our problem solving abilities. I am sure I'll be coming back to this in the coming weeks. 

I am starting to think ChatGPT can be to one what Reggie Jeeves is to Bertram Wooster...