# Introduction

This repo documents the private API used by the [Anova Precision Oven] mobile applications. This covers the Cloud API, rather than the API that the Oven uses to talk to the cloud.

The oven uses [Google Cloud IoT Core] over [mqtt] to communicate to the cloud. While MQTT is an open protocol, unfortunately it uses certificate pinning to communicate with Google which makes it impossible to view the communications between the oven and Google.

The oven firmware [1.4.24] is available but unfortunately it's encrypted as well so it's impossible to extract the keys used. Any suggestions or explorations in how to inspect the oven traffic directly are appreciated.

## Suggestions?

Open a pull request or an issue if you find a problem.

[Anova Precision Oven]: https://anovaculinary.com/products/anova-precision-oven
[Google Cloud IoT Core]: https://cloud.google.com/iot-core
[mqtt]: https://cloud.google.com/iot/docs/how-tos/mqtt-bridge
[1.4.24]: https://storage.googleapis.com/anova-app.appspot.com/oven-firmware/oven-controller-1.4.24.bin
