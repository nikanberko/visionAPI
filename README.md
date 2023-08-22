# visionAPI
package com.example.properbooker;

import com.google.cloud.vision.v1.AnnotateImageRequest;
import com.google.cloud.vision.v1.AnnotateImageResponse;
import com.google.cloud.vision.v1.BatchAnnotateImagesResponse;
import com.google.cloud.vision.v1.Feature;
import com.google.cloud.vision.v1.Feature.Type;
import com.google.cloud.vision.v1.Image;
import com.google.cloud.vision.v1.ImageAnnotatorClient;
import com.google.protobuf.ByteString;

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;

public class ProperbookerApplication {
    public static void main(String... args) throws Exception {

        try (ImageAnnotatorClient vision = ImageAnnotatorClient.create()) {


            String fileName = "src/main/java/idcard.jpg";

            Path path = Paths.get(fileName);

            byte[] data = Files.readAllBytes(path);

            ByteString imgBytes = ByteString.copyFrom(data);

            List<AnnotateImageRequest> requests = new ArrayList<>();

            Image img = Image.newBuilder().setContent(imgBytes).build();

            Feature feat = Feature.newBuilder().setType(Type.DOCUMENT_TEXT_DETECTION).build();

            AnnotateImageRequest request =
                    AnnotateImageRequest.newBuilder().addFeatures(feat).setImage(img).build();

            requests.add(request);

            BatchAnnotateImagesResponse response = vision.batchAnnotateImages(requests);

            List<AnnotateImageResponse> responses = response.getResponsesList();

            StringBuilder fullText = new StringBuilder();

            int MRZEndIndex;

            StringBuilder MRZ = new StringBuilder();

            for (AnnotateImageResponse res : responses) {
                if (res.hasError()) {
                    System.out.format("Error: %s%n", res.getError().getMessage());
                    return;
                }
                fullText.append(res.getFullTextAnnotation().toString());
            }

            //System.out.format("Error: %s%n", MRZ);
            MRZEndIndex=fullText.toString().lastIndexOf('<')+1;
            System.out.format("MRZ: %s%n", fullText.toString().substring(MRZEndIndex-94,MRZEndIndex));
           // System.out.format("PARSED MRZ: %s%n", com.innovatrics.mrz.MrzParser.parse("IOHRV113880101141043811283<<<<\n9811289M2307183HRV<<<<<<<<<<<1\nPAVKOVIC<<NIKOLA<<<<<<<<<<<<<<").toString());


        }
    }
}
