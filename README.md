Real Life Testing
===========================

Svar på spørgsmålene, og koden ligger i bunden...

### Explain the purpose of the Test (what the original test exposed, and what your test expose)

Meningen med testen er at tjekke om programmet er i stand til at genkende forskellige nummerplader ud fra en mappe med billeder. 

Der er 2 forskellige:
intelligenceSingleTest tester bare et enkelt billede “test_041.jpeg” om den kan finde nummerplanden udfra billedet
testAllSnapshots tester alle billederne der ligger i mappen /resources/snapshots. Denne test giver ikke noget helt korrekt resultat, da vi ikke tjekker om nummerpladen er det som billedet burde være..

Grunden til at de har testen er for at være sikker på at det stykke software de har skrevet virker som det er forventet. De har gjort det automatisk for at spare tid, da de kun skal skrive en enkel kommando.

### Explain about Parameterized Tests in JUnit and how you have used it in this exercise.

Parameterized tests gør det muligt kører mange tests uden at skulle skrive en masse kode. I stedet for at du skal hardcode dine værdier i X antal tests, så kan du lave et liste med det du vil teste på, samt hvad du forventer. Dette er især brugbart i det her eksempel, hvor at vi skal tjekke en masse forskellige billeder efter nummerplader. Den måde de havde gjort før hvor de talte hvor mange korrekte og forkerte de havde gør det ikke nemt at se hvilket billede det var der fejlede. Ved at bruge parameterized tests får man en meget bedre oversigt.

### Explain the topic Data Driven Testing, and why it often makes a lot of sense to read test-data from a file

Data-Driven-Testing er en måde hvorpå man får test dataen fra en ekstern part. For eksempel en fil som i projektet. Ved at bruge Data Driven Testing får man meget lettere ved at tilføje nye test cases, og derfor sikre man at produktet virker som man forventer.

###  Your answers to the question; whether what you implemented was a Unit Test or a JUnit Test, the problems you might have discovered with the test and, your suggestions for ways this could have been fixed..
Jeg vil mene at det er en JUnit Test, da det er java kode jeg har haft skrevet Man kunne have lavet det samme med xUnit i .NET ved at bruge “Theory”.

Derudover kan man evt. sige at hver af de enkelte tests er en unit test, som tester en unit i systemet - bare flere gange.

### The steps you took to include Hamcrest matchers in the project, and the difference they made for the test
Jeg tilføjede:
```
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-library</artifactId>
    <version>1.3</version>
    <scope>test</scope>
</dependency>
```

Til pom.xml og kørte `mvn clean install`. Derefter brugte ejg de korrekt imports og ændrede `assertEquals` til `assertThat(this.expected, is(equalTo((spz))))`

Jeg fik ikke noget nyt information, da jeg kørte det hele fra commandline - dog er formateringen blevet meget lettere at læse.

# Updated java code
```
/*
 * Copyright 2013 JavaANPR contributors
 * Copyright 2006 Ondrej Martinsky
 * Licensed under the Educational Community License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.osedu.org/licenses/ECL-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an "AS IS"
 * BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
 * or implied. See the License for the specific language governing
 * permissions and limitations under the License.
 */

package net.sf.javaanpr.test;

import net.sf.javaanpr.imageanalysis.CarSnapshot;
import net.sf.javaanpr.intelligence.Intelligence;
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ErrorCollector;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xml.sax.SAXException;

import javax.xml.parsers.ParserConfigurationException;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameters;

import java.util.*;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.equalTo;

import static org.junit.Assert.*;

@RunWith(Parameterized.class)
public class RecognitionAllIT {

    private static final int currentlyCorrectSnapshots = 53;
    private static final Logger logger = LoggerFactory.getLogger(RecognitionIT.class);

    @Parameters
    public static Collection<Object[]> data() throws Exception {
        String resultsPath = "src/test/resources/results.properties";
        InputStream resultsStream = new FileInputStream(new File(resultsPath));

        Properties properties = new Properties();
        properties.load(resultsStream);
        resultsStream.close();

        List<Object[]> parameters = new ArrayList<Object[]>();

        Enumeration e = properties.propertyNames();

        while (e.hasMoreElements()) {
            String key = (String) e.nextElement();

            parameters.add(new Object[] { key, properties.getProperty(key) });
        }

        return parameters;
    }

    private String fileName;

    private String expected;

    public RecognitionAllIT(String fileName, String expected) {
        this.fileName = fileName;
        this.expected = expected;
    }

    @Rule
    public ErrorCollector recognitionErrors = new ErrorCollector();

    //    TODO 3 Fix for some strange encodings of jpeg images - they don't always load correctly
    //    See: http://stackoverflow.com/questions/2408613/problem-reading-jpeg-image-using-imageio-readfile-file
    //    B/W images load without a problem: for now - using snapshots/test_041.jpg
    @Test
    public void test() throws IOException, ParserConfigurationException, SAXException {
        final String image = "snapshots/" + this.fileName;
        CarSnapshot carSnap = new CarSnapshot(image);
        assertNotNull("carSnap is null", carSnap);
        assertNotNull("carSnap.image is null", carSnap.getImage());
        Intelligence intel = new Intelligence();
        assertNotNull(intel);
        String spz = intel.recognize(carSnap);
        
        assertNotNull("The licence plate is null - are you sure the image has the correct color space?", spz);
        assertThat(this.expected, is(equalTo((spz))));
        carSnap.close();
    }

}
```
