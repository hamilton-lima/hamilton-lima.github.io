package models;

import java.util.List;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.OneToMany;

import com.avaje.ebean.Model;

@Entity
public class Product extends Model {

	@Id
	public Long id;
	public String name;
	public String barcode;
	public String description;
	
	@OneToMany(mappedBy="product")
	public List<Price> prices;

}
