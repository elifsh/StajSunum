```java
package ÇalışanVTler;

import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class PsyArXiv {

    public static void main(String[] args) {
        PsyArXiv psy = new PsyArXiv();
        psy.getPsyArXiv(20000);
    }
public void getPsyArXiv(int totalPages) {
    String startDate = "2017-01-01";
    String endDate = "2024-12-31";
    String baseUrl = "https://api.osf.io/v2/preprints/?filter[date_created][gte]=" + startDate +
            "&filter[date_created][lte]=" + endDate;
    try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
        // Tüm sayfalardan veri çek
        for (int pageNumber = 1; pageNumber <= totalPages; pageNumber++) {
            String url = baseUrl + "&page=" + pageNumber;
            try (CloseableHttpResponse response = httpClient.execute(new HttpGet(url))) {
                HttpEntity entity = response.getEntity();
                if (entity != null) {
                    String result = EntityUtils.toString(entity);
                    JSONObject json = new JSONObject(result);
                    JSONArray data = json.getJSONArray("data");
                    // Mevcut sayfadaki her preprint'i işle
                    for (int i = 0; i < data.length(); i++) {
                        JSONObject preprint = data.getJSONObject(i);
                        String id = preprint.getString("id");
                        String title = preprint.getJSONObject("attributes").getString("title");
                        String doi = preprint.getJSONObject("links").optString("html", "N/A");
                        String date = preprint.getJSONObject("attributes").getString("date_published").substring(0, 4);
                        String pdfUrl = "https://osf.io/" + id + "/download/";
                        String publisher = preprint.getJSONObject("relationships").getJSONObject("provider").getJSONObject("data").getString("id");
                        // Contributors endpoint'ine GET isteği gönder
                        String authors = getContributors(id, httpClient);
                        if (publisher.length() <= 3) {
                            publisher = authors;
                        }
                        if (!isValidPdfUrl(pdfUrl)) {
                            continue;
                        }
                        // Çekilen veriyi ekrana yazdır
                        System.out.println("ID: " + id);
//                        System.out.println("Title: " + title);
//                        System.out.println("DOI: " + doi);
//                        System.out.println("Date Published: " + date);
                        System.out.println("PDF URL: " + pdfUrl);
//                        System.out.println("Publisher : " + publisher);
//                        System.out.println("Authors: " + authors);
//                        System.out.println();
//                        System.out.println("-----------------------------------");

//                        String type = "webspider_docs2";
//                        String sourceurl = doi;
//                        String pdfurl = pdfUrl;
//                        String status = "active";
//                        String crossref = "0";
//                        String scope = "0";
//                        String attribute = "0";
//                        String folder_name = "psyarxiv/";
//                        int permission = 1;
//                        String file_name = id + ".pdf";
//                        String process_type = "Demo";
//                        String abc =  setElasticProcess(type, publisher, title, authors, date, sourceurl, status, crossref, scope, attribute, folder_name, permission, file_name, process_type, pdfurl);
//                        System.out.println("abc = " + abc);
                    }
                }
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
    private boolean isValidPdfUrl(String pdfUrl) {
        URL url;
        HttpURLConnection connection = null;
        try {
            url = new URL(pdfUrl);
            connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("HEAD");
            int responseCode = connection.getResponseCode();
            return responseCode == HttpURLConnection.HTTP_OK;
        } catch (IOException e) {
            return false;
        }
    }
    public String getContributors(String preprintId, CloseableHttpClient httpClient) throws IOException {
        StringBuilder contributorsList = new StringBuilder();
        String contributorsUrl = "https://api.osf.io/v2/preprints/" + preprintId + "/contributors/";
        CloseableHttpResponse response = null;
        try {
            HttpGet request = new HttpGet(contributorsUrl);
            response = httpClient.execute(request);
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                String result = EntityUtils.toString(entity);
                JSONObject json = new JSONObject(result);
                if (json.has("data")) {
                    JSONArray data = json.getJSONArray("data");
                    for (int i = 0; i < data.length(); i++) {
                        JSONObject contributor = data.getJSONObject(i);
                        if (contributor.getJSONObject("embeds").getJSONObject("users").has("data")) {
                            JSONObject users = contributor.getJSONObject("embeds").getJSONObject("users");
                            if (users.has("data")) {
                                String fullName = users.getJSONObject("data").getJSONObject("attributes").getString("full_name");
                                if (contributorsList.length() > 0) {
                                    contributorsList.append(", ");
                                }
                                contributorsList.append(fullName);
                            }
                        }
                    }
                }
            }
        } finally {
            if (response != null) {
                response.close();
            }
        }

        return contributorsList.toString();
    }
//    public String setElasticProcess(String type, String publisher, String title, String authors, String date, String sourceurl, String status, String crossref, String scope, String attribute, String folder_name, int permission, String file_name, String process_type, String pdfurl) {
//
//        String donus = "0";
//        JSONObject parent = new JSONObject();
//        JSONObject item = new JSONObject();
//        item.put("type", type); //Makale
//        item.put("publisher", publisher); //Makale
//        item.put("title", title); //Makale
//        item.put("authors", authors); //Makale
//        item.put("date", date); //Makale
//        item.put("sourceurl", sourceurl); //Makale
//        item.put("pdfurl", pdfurl); //Makale
//        item.put("status", status); //Makale
//        item.put("crossref", crossref); //Makale
//        item.put("scope", scope); //Makale
//        item.put("attribute", attribute); //Makale
//        item.put("folder_name", folder_name); //Makale
//        item.put("permission", permission); //Makale
//        item.put("file_name", file_name); //Makale
//        item.put("process_type", process_type); //Makale
//
//        JSONObject auth = new JSONObject();
//        auth.put("username", "amungen@gmail.com"); //Kendi Kullanıcı Adınız
//        auth.put("password", "ahmet1990"); // Kendi Şifreniz
//        auth.put("role", "engineer"); //Kendi rolünüz
//        parent.put("auth", auth);
//        parent.put("item", item);
//
//        try {
//            // setTrustAllCerts();
//        } catch (Exception e) {
//        }
//
//        try {
//            URL url = new URL("http://89.163.144.5:8080/intihalveriservice/eekle.jsp");
//            //HttpsURLConnection con = (HttpsURLConnection) url.openConnection();
//            HttpURLConnection con = (HttpURLConnection) url.openConnection();
//            con.setRequestMethod("POST");
//            con.setRequestProperty("Content-Type", "application/json; utf-8");
//            con.setRequestProperty("Accept", "application/json");
//            con.setDoOutput(true);
//
//            String jsonInputString = parent.toString();
//            try ( OutputStream os = con.getOutputStream()) {
//                byte[] input = jsonInputString.getBytes("UTF-8");
//                os.write(input, 0, input.length);
//            }
//            try ( BufferedReader br = new BufferedReader(new InputStreamReader(con.getInputStream(), "UTF-8"))) {
//                StringBuilder response = new StringBuilder();
//                String responseLine = null;
//                while ((responseLine = br.readLine()) != null) {
//                    response.append(responseLine.trim());
//                }
//                //      System.out.println(response.toString());
//                donus = response.toString();
//            }
//
//        } catch (Exception e) {
//            e.printStackTrace();
//        }
//
//        return donus;
//    }
}

```
