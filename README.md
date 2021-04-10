# Introduction to jq
Learn how to get started with [jq](https://stedolan.github.io/jq/)!

**We recommend going through this walkthrough in [Gitpod](https://gitpod.io/) as Gitpod will have everything we need for this walkthrough. Hit the button below to get started!**
</br>
</br>
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/Anant/example-introduction-to-jq) 
</br>
</br>
If you prefer to do this locally, then you may need to download the latest version of [jq](https://stedolan.github.io/jq/) if you do not have it on your local OS.

## **1. Basic Printing**

### **1.1 - Print all**
The simplest jq program is the expression `.`, which takes the input and outputs it as is.

**1.1.1 - Print using input file `cars.json`.**

```bash
jq '.' cars.json 
```

**1.1.1 - Print using output of pipe.**

```bash
cat cars.json | jq '.'
```

### **1.2 - Length of JSON .**

```bash
jq 'length' cars.json 
```

## **2. Select Object**

### **2.1 - Print first object.**

```bash
jq '.[0]' cars.json
```

### **2.2 - Print 3rd and 5th objects.**

```bash
jq '.[2], .[4]' cars.json
```

## **3. Print Keys and Values**

### **3.1 - Print Keys.** 

```bash
jq '.[] | keys' cars.json
```

### **3.2 - Select value of key of objects nested in array**

```bash
jq '.[].id' cars.json
```

### **3.3 - Select values of multiple keys**

**3.3.1 - Top level** 

```bash
jq '.[] | .id, .location' cars.json
```

**3.3.2 - Nested** 

```bash
jq '.[].car | .make, .model, .year' cars.json
```

## **4. Change values**

```bash
jq '.[].location |= "Virginia Beach"' cars.json 
```

## **5. Select based on filters**

### **5.1 - Based on make** 

```bash
jq '.[].car | select(.make == "Buick")' cars.json 
```

### **5.2 - Based on year**

```bash
jq '.[].car | select(.year > 2000)' cars.json 
```

## **6. Delete Keys**

### **6.1 - Delete Single Key**

```bash
jq 'del(.[].id)' cars.json
```

### **6.2 - Delete multiple nested keys.** 

```bash
jq 'del(.[].car.year) | del(.[].car.make)' cars.json
```


## **7. Potential Real World Application**
The next part of the walkthrough will focus on a scenario where someone could `jq` for data engineering/wrangling in a "real world setting". At the time of creating this repo, the 2021 MLB season has just started up. Nowadays, data is heavily ingrained into professional sports, but baseball especially with [Sabermetrics](https://en.wikipedia.org/wiki/Sabermetrics) (list of just [offensive statistics](https://library.fangraphs.com/offense/offensive-statistics-list/)). Moving forward, we will use `jq` to do some data engineering/wrangling and give our analysts some basic statistics to work with. 

### **7.1 - Get roster of the Phillies**

```bash
curl 'http://lookup-service-prod.mlb.com/json/named.roster_40.bam?team_id='143'' | jq '.roster_40.queryResults.row' > phillies.json
```

### **7.2 - Sort / Group By** 
We could sort or group by to do some filtering; however, we don't use it moving forward. Here is what those options could look like though.


**7.2.1 - Sort by position.** 
sort_by(foo) compares two elements by comparing the result of foo on each element.

```bash
jq 'sort_by(.position_txt)' phillies.json
```

**7.2.2 - Group by position.** 
group_by(.foo) takes as input an array, groups the elements having the same .foo field into separate arrays, and produces all of these arrays as elements of a larger array, sorted by the value of the .foo field.

```bash
jq 'group_by(.position_txt)' phillies.json
```

### **7.3 - Filter out pitchers** 
`-s`: Read the entire input stream into a large array
```bash
jq '.[] | select(.position_txt == "P")' phillies.json | jq -s '.' > pitchers.json
```

### **7.4 - Create pitchers CSV** 
The script file `toCSV.jq` does the following:
1. Take the array input containing all the different keys and set them as the columns
2. For each object in the array input, map the columns to the corresponding values in the object and set them as the rows.
3. Put the column names before the rows, as a header for the CSV, and pass the resulting row stream to the @csv filter.
4. Use the -r flag to get the result as a raw string

```bash
jq -r -f toCSV.jq pitchers.json > pitchers.csv
```

### **7.5 - Get stats for the pitchers in the rotation who are either on our team or we are facing** 

**7.5.1 - Aaron Nola (SP).** 

```bash
curl 'http://lookup-service-prod.mlb.com/json/named.sport_career_pitching.bam?league_list_id=%27mlb%27&game_type=%27R%27&player_id=%27605400%27' | jq '.sport_career_pitching.queryResults.row' > nola.json
```

**7.5.2 - Zack Wheeler (SP2).** 

```bash
curl 'http://lookup-service-prod.mlb.com/json/named.sport_career_pitching.bam?league_list_id=%27mlb%27&game_type=%27R%27&player_id=%27554430%27' | jq '.sport_career_pitching.queryResults.row' > wheeler.json
```

**7.5.3 - Zach Eflin (SP3).** 

```bash
curl 'http://lookup-service-prod.mlb.com/json/named.sport_career_pitching.bam?league_list_id=%27mlb%27&game_type=%27R%27&player_id=%27621107%27' | jq '.sport_career_pitching.queryResults.row' > eflin.json
```

### **7.6 - Add their names to their stats** 

**7.6.1 - Aaron Nola (SP).** 

```bash
jq '.full_name = "Aaron Nola"' nola.json > nola.json.tmp && mv nola.json.tmp nola.json 
```

**7.6.2 - Zack Wheeler (SP2).** 

```bash
jq '.full_name = "Zack Wheeler"' wheeler.json > wheeler.json.tmp && mv wheeler.json.tmp wheeler.json 
```

**7.6.3 - Zach Eflin (SP3).** 

```bash
jq '.full_name = "Zach Eflin"' eflin.json > eflin.json.tmp && mv eflin.json.tmp eflin.json 
```

### **7.7 - Combine their data into one JSON array** 
```bash
jq -s '.' nola.json eflin.json wheeler.json > rotation.json
```

### **7.8 - Generate rotation CSV file** 
```bash
jq -r -f toCSV.jq rotation.json > rotation.csv
```

We can then hand this CSV back to our analytics team and they can then use it to potentially gameplan for the series with / against these pitchers. With that, we will wrap up our walkthrough on basic `jq` operations as well as a potential real-world scenario in which we can use a tool like `jq` to do some fast data engineering/wrangling.

### Additional Resources
- [Live Demo]()
- [Accompanying Blog]()
- [Accompanying SlideShare]()
- https://stedolan.github.io/jq/manual/
