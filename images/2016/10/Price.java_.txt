package models;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

import com.avaje.ebean.Model;
import com.fasterxml.jackson.annotation.JsonIgnore;

@Entity
public class Price extends Model {

	@Id
	public Long id;
	public Double value;
	public String currency;

	@ManyToOne
	public Store store;

	@ManyToOne
	@JsonIgnore
	public Product product;
}
