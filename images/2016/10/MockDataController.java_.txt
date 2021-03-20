package controllers;

import java.util.List;
import java.util.Random;

import models.Price;
import models.Product;
import models.Store;
import play.libs.Json;
import play.mvc.Controller;
import play.mvc.Result;
import play.twirl.api.Content;

import com.avaje.ebean.Model;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class MockDataController extends Controller{

	private static final String DEFAULT_CURRENCY = "CAD";
	private static final double BASE_VALUE = 100.0;
	
	static String products [] = {"A","B","C","D"};
	static String stores [] = {"store 1","store 2","store 3","store 4"};
	
	static double latitude = -22.884077;
	static double longitude = -43.281055;
	
	static double latitude2 = -22.882859;
	
	// to distribute the stores
	static double latitudeDiff = latitude2 - latitude; 
	
	@SuppressWarnings({ "rawtypes", "unchecked" })
	public Result add() {

        List<Product> found = new Model.Finder(Product.class).all();
        if( found.size() > 0 ){
            return ok( getPrettyJson(found));
        }

    	for (int i = 0; i < products.length; i++) {
			Product product = new Product();
			product.name = products[i];
			product.barcode = pad(i, 5);
			product.save();
			
			for (int j = 0; j < stores.length; j++) {
				Store store = getStore(stores[j]);
				store.latitude = latitude + (j * latitudeDiff);
				store.longitude = longitude;
				store.save();
				
				Price price = new Price();
				price.currency = DEFAULT_CURRENCY;
				Random r = new Random();
				price.value = round(r.nextDouble() * BASE_VALUE); 
				price.store = store;
				price.product = product;
				price.save();
				
				product.prices.add(price);
			}

			product.save();
		}

        found = new Model.Finder(Product.class).all();
        return ok( getPrettyJson(found));
    }

	private Double round(double d) {
		return (double) Math.round(d * 100) / 100;
	}

	private String getPrettyJson(Object input) {
        JsonNode node = Json.toJson(input);
		try {
	        ObjectMapper mapper = new ObjectMapper(); 
	        String pretty;
			pretty = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(node);
			return pretty;
		} catch (JsonProcessingException e) {
			return node.toString();
		}
	}

	private String pad(int value, int repeat) {
		StringBuffer buffer = new StringBuffer();
		for (int j = 0; j < repeat; j++) {
			buffer.append(value);
		}
		return buffer.toString();
	}

	private Store getStore(String name) {
		Store result = (Store) new Model.Finder(Store.class).where().eq("name", name).findUnique();
		if( result == null ){
			result = new Store();
			result.name = name;
		}
		return result;
	}


}
