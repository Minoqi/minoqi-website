---
title: CastleDB Cleaner - CastleDB to JSON Converter
published: 2025-01-29
description: 'A single python script that converts CastleDB .cdb file to an easier to use JSON file'
image: ''
tags: [Python, Devblog, Tutorial]
category: ''
draft: false 
lang: ''
---

![CastleDB Cleaner Promo Img](src/assets/images/castledb_cleaner/CastleDBCleanerPromoImg.png)

Recently, I was on the hunt for a database system to handle large amount of game data outside of the engine. I knew I wanted it to use the JSON file system and wanted it to be in a grid format. Unforuntaley, my options were limited but I decided to go with [CastleDB](http://castledb.org/). 

All was going well until I took a look at the JSON file, everything was stored in an array! I tried [Depot](https://depot-editor.com/) (VS Code plugin) which does the same thing! It drove me crazy, as I fail to see why that's how it would be stored over a typical dictionary with ID format. If you don't use that method you loose a lot of the benefits of using a dictionary system. 

But I thought "Wait... I could just convert it no?" and now here we are. I've made a quick python script that converts each sheet from `CastleDB` into it's own JSON file using a unique ID systme instead, thus making it much easier and nicer to use in my games. You can view it on my github [here](https://github.com/Minoqi/CastleDB-Cleaner/tree/main).

---

# How Do You Use It?

The program is quite easy to use as it's a single script. You just need `.cdb` file and somewhere to store the output. Here's a brief overview of the steps of the program:

1. Run the `ConvertCastleDBToJSON.py` file
2. Paste in the file path of the `.cdb` file
3. If you want to store the output in the same location that the python script is in, say `y` otherwise say `n`
4. Otherwise, if you want to store the output in the same location that the `.cdb` file is in, say `y` otherwise say `n`
5. Otherwise, paste in the file path you want to save the output to
6. If the file output already exists, say `y` for it to be overwritten or say `n` to skip it
7. Enter in the name of the column that stores the ID value. Careful how you write it, it has to be exact!

And that's it! It'll print out a message after each sheet is done and then a final message when it's completely done. Quite simple!

![CastleDB Cleaner console output screenshot](src/assets/images/castledb_cleaner/ConsoleOutput.PNG)

---

# How Does It Work?

While this won't be as detailed of a description as my usual tutorials, it is detailed enough that you should be able to understand the code and adjust it to fit your needs if you so desire. First let's start with taking in the `.cdb` file.

- The program asks you to paste in the location of the `.cdb` file
- It removes any accidental whitespace and then checks to make sure the file exists
- If it doesn't we quit out the program

```python
# Get the files paths
fileToConvert = input("Enter the Castle DB file path: ").strip()


# Open and load in the original file
try:
    with open(fileToConvert, "r") as file:
        originalData = json.load(file)
    print("SUCCESSFULLY LOADED")
except FileNotFoundError:
    print("ERROR: File not found")
    quit()
except json.JSONDecodeError:
    print("ERROR: Invalid JSON format")
    quit()
except Exception as e:
    print(f"ERROR: {e}")
    quit()
```

- Next we get the users output destination
- There are a few different methods to help speed up the process so there are a few options it needs to go through
- It's important we make sure the file path has the final `\` at the end of it, otherwise it'll save in the folder above it, we can use pythons `os.sep` to solve this

```python
# Get the destination for converted file
useCurrentPath = input("Do you want to store the converted files in the same location this script is located in? (y/n) ").strip()
while useCurrentPath.lower() != "y" and useCurrentPath.lower() != "n":
    print("ERROR: Asnwer given is invalid, please try again.")
    useCurrentPath = input("Do you want to store the converted files in the same location this script is located in? (y/n) ").strip()

if useCurrentPath.lower() == "n":
    useCastleDBPath = input("Do you want to store the converted files in the same location the CastleDB file is located in? (y/n) ").strip()
    while useCastleDBPath.lower() != "y" and useCastleDBPath.lower() != "n":
        print("ERROR: Asnwer given is invalid, please try again.")
        useCastleDBPath = input("Do you want to store the converted files in the same location the CastleDB file is located in? (y/n) ").strip()

if useCurrentPath.lower() == "y":
    locationToSendTo = os.getcwd()
elif useCastleDBPath.lower() == "y":
    locationToSendTo = os.path.dirname(fileToConvert)
else:
    locationToSendTo = input("Enter the file path for the folder you want the conversion stored in: ").strip()

    while os.path.exists(locationToSendTo) == False:
        print("ERROR: That file path does not exist, please try again")
        locationToSendTo = input("Enter the file path for the folder you want the conversion stored in: ").strip()

# Add trailing slash if it's not there
if not locationToSendTo.endswith(os.sep):
    locationToSendTo += os.sep
```

- Then we get any leftover information
- At the time of writing this this includes whether or not the file should be overwritten and what the ID is

```python
# Get any other necessary information
overrideSameName = input("If the file already exists should it overrite it? (y/n) ").strip()

while overrideSameName.lower() != "y" and overrideSameName.lower() != "n":
    print("ERROR: Answer given is invalid, please try again.")
    overrideSameName = input("If the file already exists should it overrite it? (y/n) ").strip()

if overrideSameName.lower() == 'y':
    overrideEnabled = True

idString = input("What's the name of the key that's storing the ID value? Keep in mind this is case sensitive! -> ")
```

- Last but not least we actually sort through and convert the `.cdb` file
- We use a for loop to go through each sheet
- Then we make sure the ID exists and if it doesn't we skip that sheet
- From there we loop through each line of the sheet and store the data into a dictionary
    - I added in an extra line that will remove the ID column from the final dictionary as it's not needed!
- Once that's done we convert the dictionary into a `json` file and save it at the desired location, using the sheet name as the `json` files name
- Lastly we empty the `convertedData` dictionary to prepare for the next round

```python
# Convert each sheet
for sheet in originalData["sheets"]:
    # Make sure ID exists, otherwise skip sheet
    if idString not in sheet["lines"][0]:
        print(f"ERROR: ID not found in file, skipping sheet {sheet["name"]}... (Given ID: {idString})")
        continue

    # Save to it's own file
    for line in sheet["lines"]:
        convertedData[line[idString]] = line
        del convertedData[line[idString]][idString]
    
    finalPathName = locationToSendTo + sheet["name"] + ".json"
    
    if os.path.exists(finalPathName):
        if overrideEnabled == False:
            print(f"ERROR: File aready exists and override is set to false, skipping... ({finalPathName})")
            continue
        else:
            os.remove(finalPathName)
        
    with open(finalPathName, "w") as file:
        json.dump(convertedData, file, indent=4)
        print(f"File Done: {finalPathName}")
    
    # Reset
    convertedData = {}
```

And that's it! Quite simple huh? I could probably create the same thing for `Depot` but `CastleDB` let's you go back and change the type of data in a column so I plan on sticking with it, but `Depot` is a nice alternative!