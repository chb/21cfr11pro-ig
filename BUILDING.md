## Building the IG

  1. Set the JRE to use to 1.8: `jenv shell 1.8` (or similar)
  2. Install ruby (if it isnt aready) `brew install ruby`
  3. Install jekyll `sudo gem install jekyll` 

Then launch the publisher. Make sure the ig.json has a version that supports this line in ig.json:

```
  "version" : "4.0.1"
```

`java -jar ~/bin/jars/org.hl7.fhir.publisher.jar -ig ig.json`

