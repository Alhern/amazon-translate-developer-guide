# Using Amazon Translate to Translate Large Documents<a name="examples-split"></a>

You can split large documents into smaller parts to keep the total document size below the document size limit\. For more information about document size limits, see [Limits](beta-limits-guidelines.md#beta-limits)\. The following Java program breaks long text documents into individual sentences and then translates each sentence from the source language to the target language\. The program contains two sections:

+ The `SentenceSegmenter` class that is responsible for breaking the source string into individual sentences\. The sample uses the International Components for Unicode \(ICU\) `BreakIterator` class\. You can use the Java `BreakIterator` class as an alternative\. For more information, see [Using the Java BreakIterator Class](#split-java-iterator)\.

+ The `main` function that calls the `Translate` operation for each sentence in the source string\. The `main` function also handles authentication with Amazon Translate\.

**To configure the example**

1. Install and configure the AWS SDK for Java\. For instructions for installing the SDK for Java, see [ Set up the AWS SDK for Java](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-install.html)\.

1. Download and install the ICU jar files from [http://site\.icu\-project\.org/download](http://site.icu-project.org/download)\. If you are using Maven, set up dependencies from [https://mvnrepository\.com/artifact/com\.ibm\.icu/icu4j](https://mvnrepository.com/artifact/com.ibm.icu/icu4j)\.

1. Create an IAM user with the minimum required permissions to run this example\. For information about creating an IAM user, see [ Creating an IAM User in Your AWS Account ](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) in the *AWS Identity and Access Management User Guide*\. For the required permissions policies, see [Using Identity\-Based Polices \(IAM Policies\) for Amazon Translate](access-control-managing-permissions.md)\.

1. Set up the credentials needed to run the sample\. For instructions, see [ Set up AWS Credentials and Region for Development ](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) in the *AWS SDK for Java Developer Guide*\.

1. Create a new project in your Java IDE and copy the source code\.

1. Change the region to the region where you want to run the Amazon Translate operation\. For a list of supported regions for Amazon Translate, see [Beta Guidelines and Limits](beta-limits-guidelines.md)\. 

1. Change the source and target languages to the languages to translate between\.

1. Run the sample to see the translated text on standard output\.

```
import com.amazonaws.auth.AWSCredentialsProviderChain;
import com.amazonaws.auth.EnvironmentVariableCredentialsProvider;
import com.amazonaws.auth.SystemPropertiesCredentialsProvider;
import com.amazonaws.auth.profile.ProfileCredentialsProvider;
import com.amazonaws.services.translate.AmazonTranslate;
import com.amazonaws.services.translate.AmazonTranslateClient;
import com.amazonaws.services.translate.model.TranslateTextRequest;
import com.amazonaws.services.translate.model.TranslateTextResult;
import com.ibm.icu.text.BreakIterator;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class MultiSentenceTranslator {

    public static void main(String[] args) {
        // Define the text to be translated here
        String region = "region";
        String text = "Text to be translated";

        String sourceLang = "source language";
        String targetLang = "target language";

        // Break text into sentences
        SentenceSegmenter sentenceSegmenter = new SentenceSegmenter();
        List<String> sentences = new ArrayList<>();
        try {
            sentences = sentenceSegmenter.segment(text, sourceLang);
        } catch (Exception e) {
            System.out.println(e);
            System.exit(1);
        }

        // Create credentials using a provider chain that will evaluate in order;
        // a) Any Java system properties
        // b) Any environment variables
        // c) Any profile file
        AWSCredentialsProviderChain DefaultAWSCredentialsProviderChain = new AWSCredentialsProviderChain(
                new SystemPropertiesCredentialsProvider(),
                new EnvironmentVariableCredentialsProvider(),
                new ProfileCredentialsProvider()
        );

        // Create an Amazon Translate client
        AmazonTranslate translate = AmazonTranslateClient.builder()
                .withCredentials(DefaultAWSCredentialsProviderChain)
                .withRegion(region)
                .build();

        // Translate sentences and print the results to stdout
        for (String sentence : sentences) {
            TranslateTextRequest request = new TranslateTextRequest()
                    .withText(sentence)
                    .withSourceLanguageCode(sourceLang)
                    .withTargetLanguageCode(targetLang);
            TranslateTextResult result = translate.translateText(request);
            System.out.println("Original text: " + sentence);
            System.out.println("Translated text: " + result.getTranslatedText());
        }
    }

}

class SentenceSegmenter {
    public List<String> segment(final String text, final String lang) throws Exception {
        List<String> res = new ArrayList<>();
        BreakIterator sentenceIterator = BreakIterator.getSentenceInstance(new Locale(lang));
        sentenceIterator.setText(text);
        int prevBoundary = sentenceIterator.first();
        int curBoundary = sentenceIterator.next();
        while (curBoundary != BreakIterator.DONE) {
            String sentence = text.substring(prevBoundary, curBoundary);
            res.add(sentence);
            prevBoundary = curBoundary;
            curBoundary = sentenceIterator.next();
        }
        return res;
    }

}
```

## Using the Java BreakIterator Class<a name="split-java-iterator"></a>

Depending on the language pair and your use case, the Java `BreakIterator` class may provide better performance\. You should not use it if the source language is Arabic \(ar\)\. If you want to use the Java `BreakIterator`, make the following changes to the sample:

1. Remove the following line from the imports section of the sample:

   ```
   import com.ibm.icu.text.BreakIterator;
   ```

1. Add the following line to the imports section of the sample:

   ```
   import java.text.BreakIterator;
   ```

The sample will now use the Java `BreakIterator` class\.