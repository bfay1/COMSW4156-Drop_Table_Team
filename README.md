# Drop Table Team

## COMSW4156-Software-Group

## Team Members Information:

| Name             | UNI     |
| :--------------- | :------ |
| Brendan Fay      | jbf2173 |
| Ke Liu           | kl3554  |
| Siddharth Ananth | sa4287  |
| Tony Faller      | af3370  |

## Project setup

This project uses linux-based commands to setup all the external libraries / apis needed. You would either pull this project on a linux machine or run it using WSL.

Setting up wsl: https://learn.microsoft.com/en-us/windows/wsl/install

Runnning wsl on VSCode: https://code.visualstudio.com/docs/remote/wsl

This project uses the mongocxx libraries as well as boost. To install all mongocxx dependencies, go into the project folder and run the following command: `sudo ./setup-mongocxx`. To install boost run the command `sudo apt-get install libboost-all-dev`.

This project uses Crow to set up an HTTP server to host the service. You can download the latest release of Crow [here](https://github.com/CrowCpp/Crow/releases/tag/v1.0+5) - the `.deb` version is for Ubuntu. Install with: `sudo apt install ./crow-v1.0+5.deb`.

This project uses libcurl to perform REST operations. Install this with: `sudo apt-get install libcurl4-openssl-dev`.

This project uses nlohmann json to serialize and deserialize data into json strings. Install this with: `sudo apt-get install nlohmann-json3-dev`.

This project depends on poppler. Install this with: `sudo apt-get install libpoppler-dev poppler-utils && sudo apt-get install -y libpoppler-cpp-dev`.

This project uses cpplint as a style checker. Install this with: `sudo apt install cpplint` and run with `cpplint include/*.h src/*.cpp`. Note that the tests directory was excluded from style checking because it is for internal use only, and sometimes requires format errors which otherwise should not exist in the codebase.

## Getting Started

1. In the project directory, run `make && ./bin/main` to compile the source code and run the server
2. In your web browser, visit [http://localhost:18080/translate/?tbt=hello&tl=jp&fl=en](http://localhost:18080/translate/?tbt=hello&tl=jp&fl=en)

For accessing translation, use the following query parameters:

- `tbt` = The to-be-translated text
- `tl` = The language to translate to
- `fl` = The source language of `tbt`

tl and fl should be selected from the language list in API doc.

## CI
This respository uses GitHub Actions for its Continuous Integration (CI). There are two GitHub Actions; one runs the unit tests (1) every 12 hours on `main`, (2) on every push to `main`, and (3) for every PR to `main`. The other runs `cppcheck` as a static analysis tool (1) every other day on `main`, (2) on every push to `main`, and (3) on every PR to `main`.

The static analysis action uploads the analysis artifact to the Actions tab after every run. It can be found by selecting `Actions` --> `Drop Table Team Service Static Analysis` --> Job of choice --> `cppcheck-result`.

## Branch Coverage
To manually run branch coverage, install `lcov` with this: `sudo apt-get install lcov`. Then, navigate to the `test` directory and run `./runGtest.sh`. Branch coverage will be automatically run as part of the unit tests (and therefore, as part of the CI). 

The branch coverage output can then be found in `./test/testMain.dir_out/index.html` or, in the automated case, as an artifact in GitHub Actions.

Since branch coverage analyzes which code paths are flexed by the unit tests, we do not meet the 85% coverage requirement. However, upon closer inspection, the reason for this failure is because we do not have any unit tests for `src/ocr.cpp` or `src/pdftotext.cpp`. However, these functionalities are not essential for either our service or our client application, so we've made the decision to skip implementing these unit tests in favor of focusing on the remainder of the project.

Sources:
* [The ReadME Project](https://github.com/readme/guides/sothebys-github-actions?fbclid=IwAR0P4vhynavWx4OGXc6ErreHWuE3jI7kdoPnaMCgZU2S6slIj38TBV7CFYI)
* [GitHub Documentation](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions?fbclid=IwAR3pq7lW-xNw5jzPhpfNRXhLbMIykoTBs-U6UUqAurZK8JNLUVk1zB-arGY)
* [Uploading Artifacts](https://github.com/marketplace/actions/upload-a-build-artifact)
## Endpoints:

### /translate/

- translates text from one language to another and returns output

#### Required Params:

tbt: String of the text to be translated
tl: Code for languge to be translated to
fl: Code for language to be translated from

#### Example successful response:

- textToBeTranslated: the-quick-fox-jumps-over-the-lazy-brown-dog, toLang: spa, fromLang: en Translated text: {"from":"en","to":"spa","trans_result":[{"src":"the-quick-fox-jumps-over-the-lazy-brown-dog","dst":"El zorro \u00e1gil se salta al perro marr\u00f3n perezoso"}]}

#### HTTP Error Codes

- 400 - Invalid or missing params

### /create_client/

- creates a "user" which stores the 10 most recent translations posted

#### Example successful response:

Output Message: 6531acf31b61c9766809cac1 (User id)
Output Code: 200

#### HTTP Error Codes

- 400 - Invalid or missing params

#### MongoDB error codes

- 200: successful operation
- 500: internal error

### /delete_client/

- delete user with given id (deletes all translation history)

#### Required Params:

- id: string of id returned from create_use

#### Example successful response:

Output Message: Delete Successful
Output Code: 200

#### HTTP Error Codes

- 400 - Invalid or missing params

#### MongoDB error codes

- 200: successful operation
- 404: resource not available
- 500: internal error

### /post_translation_to_client/

- puts translation in a database given a user id (returned from create_user)

#### Required Params:

- tbt: String of the text to be translated
- tl: Code for languge to be translated to
- fl: Code for language to be translated from
- id: string of id returned from create_user

#### Example successful response:

textToBeTranslated: dog, toLang: de, fromLang: en
Translated text: {"from":"en","to":"de","trans_result":[{"src":"dog","dst":"Hund"}]}
Mongo res: Update successful
Mongo code: 200

#### HTTP Error Codes

- 400 - Invalid or missing params

#### MongoDB error codes

- 200: successful operation
- 404: resource not available
- 500: internal error

### /pdf_to_text/

- scans pdf for text in some language and gives the user that text as a string

#### Required Params:

file: file in pdf format

#### Example successful response:

Output Message: content of the PDF  
Output Code: 200

text: Lorem Ipsum is simply dummy text of the printing and typesetting industry.

#### HTTP Error Codes

- 400 - Invalid or missing params

### /translate_pdf_text/

- translates text in a pdf and returns the trnslation as a string

### Required Params:

- file: PDF file as multipart form data
- tl: language to be translated to
- fl: language to be trnaslated from

### Example Successful Response

- 200 OK: source and target languages, and source and target text. may contain unicode or other characters

textToBeTranslated: Lorem Ipsum is simply dummy text of the printing and typesetting industry., toLang: de, fromLang: en
Translated text: {"from":"en","to":"de","trans_result":[{"src":"[omitted, see above]","dst":"lorem - ipsum - es - simple - Dummy - texto - de - la - impresi\u00f3n - y - tipseting - industria"}]}

### /image_to_text/

- scans image for text, similar to /pdf_to_text/

### Required Params

- file: Image file - PNG, JPG, JPEG, all formats accepted by the tesseract OCR library:

### Example Successful Response

- 200 OK: Content of Image text - may be random if image has insufficient text

text: Lorem Ipsum is simply dummy text of the printing and typesetting industry.

#### HTTP Error Codes

- 400 - Invalid or missing params

### /translate_image_text/

- translates text in an image and returns the translation as a string

### Required Params

- file: Image file in multipart form data - PNG, JPG, JPEG, all formats accepted by the tesseract OCR library:
- tl: target language
- fl: source language

### Example Successful Response

- 200 OK: source and target languages, and source and target text

textToBeTranslated: Lorem Ipsum is simply dummy text of the printing and typesetting industry., toLang: de, fromLang: en
Translated text: {"from":"en","to":"de","trans_result":[{"src":"[omitted, see above]","dst":"lorem - ipsum - es - simple - Dummy - texto - de - la - impresi\u00f3n - y - tipseting - industria"}]}

#### HTTP Error Codes

- 400 - Invalid or missing params

### /post_pdf_translation/

- puts pdf translation into database

### Required Params

- file: PDF file in multipart form data
- tl: target language
- fl: source language
- id: string of id returned from create_user

### Example Successful Response

- 200 OK: source and target languages, and source and target text

textToBeTranslated: Lorem Ipsum is simply dummy text of the printing and typesetting industry., toLang: de, fromLang: en
Translated text: {"from":"en","to":"de","trans_result":[{"src":"[omitted, see above]","dst":"lorem - ipsum - es - simple - Dummy - texto - de - la - impresi\u00f3n - y - tipseting - industria"}]}
Mongo res: Update successful
Mongo code: 200

#### HTTP Error Codes

- 400 - Invalid or missing params

#### MongoDB error codes

- 200: successful operation
- 404: resource not available
- 500: internal error

### /post_image_translation/

- puts image translation into database

### Required Params

- file: Image file in multipart form data - PNG, JPG, JPEG, all formats accepted by the tesseract OCR library:
- tl: target language
- fl: source language
- id: string of id returned from create_user

### Example Successful Response

- 200 OK: source and target languages, and source and target text

textToBeTranslated: Lorem Ipsum is simply dummy text of the printing and typesetting industry., toLang: de, fromLang: en
Translated text: {"from":"en","to":"de","trans_result":[{"src":"[omitted, see above]","dst":"lorem - ipsum - es - simple - Dummy - texto - de - la - impresi\u00f3n - y - tipseting - industria"}]}
Mongo res: Update successful
Mongo code: 200

#### HTTP Error Codes

- 400 - Invalid or missing params

#### MongoDB error codes

- 200: successful operation
- 404: resource not available
- 500: internal error

## Sources:

### Crow

- [Crow Github](https://github.com/CrowCpp/Crow)

- [Crow Tutorial](https://crowcpp.org/master/getting_started/your_first_application/)

- [Query strings](https://crowcpp.org/master/guides/query-string/)

### Curl

- [Saving CURL response](https://stackoverflow.com/questions/9786150/save-curl-content-result-into-a-string-in-c)

### Datastore

- [MongoDB Atlas Tutorials](https://mongocxx.org/mongocxx-v3/tutorial/)

- [MongoDB Connection String](https://www.mongodb.com/docs/guides/atlas/connection-string/)

- [MongoDB C++ Driver](https://www.mongodb.com/docs/drivers/cxx/)

### API

- [baidufanyi-API](https://fanyi-api.baidu.com/doc/11)

### JSON Serialization
- [nlohmann json](https://github.com/nlohmann/json)

### Tesseract

- [Tesseract](https://github.com/tesseract-ocr/tesseract)

### Environment

- [Makefile Implementation](https://github.com/evanugarte/mongocxx-tutorial/blob/09dc4bf76d57fe40cf7154a8eb9e7530d49ab536/Makefile) -- Makefile was initially used to get flags to compile MongoDB code. Modified once more dependencies were added.

- [Setup dependencies file source](https://github.com/evanugarte/mongocxx-tutorial/blob/09dc4bf76d57fe40cf7154a8eb9e7530d49ab536/setup-mongocxx) -- Modifications were made based on different versions of mongocxx (source file dowloaded older versions of mongocxx with outdated links)

## Translation API doc

### Languages list

| Supported Languages | Code |
| :------------------ | :--- |
| English             | en   |
| Japenese            | jp   |
| Korean              | kor  |
| French              | fra  |
| Spanish             | spa  |
| Thai                | th   |
| Arabic              | ara  |
| Russian             | ru   |
| Portuguese          | pt   |
| German              | de   |
| Italian             | it   |
| Greek               | el   |
| Dutch               | nl   |
| Polish              | pl   |
| Bulgarian           | bul  |
| Estonian            | est  |
| Danish              | dan  |
| Finnish             | fin  |
| Czech               | cs   |
| Romanian            | rom  |
| Slovenia            | slo  |
| Swedish             | swe  |
| Hungarian           | hu   |
| Simplified Chinese  | zh   |
| Traditional Chinese | cht  |
| Vietnamese          | vie  |

automatic detection : 'auto' can only be used in source language.

### Error code list for translation api:

| Error Code | Meaning                                 | Solution                                                                                                                                                                   |
| :--------- | :-------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0          | Success                                 |                                                                                                                                                                            |
| 52001      | Request timeout                         | Retry                                                                                                                                                                      |
| 52002      | System error                            | Retry                                                                                                                                                                      |
| 52003      | Unauthorized user                       | Please check whether your appid is correct or whether the service is enabled                                                                                               |
| 54000      | The required parameter is null          | Check whether parameters are not transmitted                                                                                                                               |
| 54001      | Signature error                         | Please check your signature generation method                                                                                                                              |
| 54003      | Limited access frequency                | Please reduce the frequency of your calls                                                                                                                                  |
| 54004      | Insufficient account balance            | Please go to the Admin console to top up your account                                                                                                                      |
| 54009      | Language detection failure              | Not in the range of languages supported                                                                                                                                    |
| 58000      | The IP address of the client is invalid | Check whether the IP address filled in the personal information is correct, you can go to the management control platform to modify, IP restrictions, IP can be left blank |
| 58002      | The service is currently closed         | Please go to the administrative console to start the service                                                                                                               |

## Postman Workspace

You can find our postman workspace here: https://comsw4156-drop-table-team.postman.co/workspace/COMSW4156-DROP-TABLE-TEAM~49856fcb-f206-4b50-b334-2d072ec3ae61/collection/25922238-41aab0aa-1091-4b54-93b7-b5d71415617d?action=share&creator=25922238

---
