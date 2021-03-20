package controllers;

import java.util.List;

import models.Product;
import models.Store;
import play.libs.Json;
import play.mvc.Controller;
import play.mvc.Result;

import com.avaje.ebean.Model;

public class ProductController extends Controller{

    @SuppressWarnings({ "rawtypes", "unchecked" })
	public Result getAll() {
        List<Product> products = new Model.Finder(Product.class).all();
        return ok( Json.toJson(products));
    }

    public Result findByBarcode(String barcode){
		Product result = (Product) new Model.Finder(Product.class).where().eq("barcode", barcode).findUnique();
        return ok( Json.toJson(result));
    }

}
