package models;

import javax.persistence.Entity;
import javax.persistence.Id;

import com.avaje.ebean.Model;

@Entity
public class Store extends Model {

	@Id
	public Long id;
	public String name;
	public Double latitude;
	public Double longitude;

}
