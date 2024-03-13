# Como resolver el error de multiples foreign keys usando sqlalchemy en FastAPI

### 1.- Vemos como se nos muestra el error:

```http
{
  "error": "Could not determine join condition between parent/child tables on relationship 
  Place.registers - there are multiple foreign key paths linking the tables.  Specify the 
  'foreign_keys' argument, providing a list of those columns which should be counted as 
  containing a foreign key reference to the parent table."
}
```

### 2.- Entendemos el error y ubicamos la solución

En un inicio es una tabla que tiene muchas claves foraneas y la idea es que cada table debe estar compuesta por una relación independiente. La solución es que cada `relationship` deba ser especificada y diferente. Y si tiene doble FK de una misma tabla, esta ultima debe especificar a que FK esta yendo.

```python
class Place(Base):
    
    __tablename__ = "places"
    idPlace = Column(Integer, primary_key=True)
    namePlace = Column(String(30))

    registers3 = relationship("Register", back_populates="place_owner", foreign_keys="[Register.idPlaceOrigin]")
    registers4 = relationship("Register", back_populates="place_owner2", foreign_keys="[Register.idPlaceDestination]")


class Register(Base):
    
    __tablename__ = "registers"
    
    idRegister = Column(Integer, primary_key=True)
    timeRegister = Column(DateTime(timezone=True), server_default=func.now())
    detail = Column(String(50))
    
    idCarReg = Column(Integer, ForeignKey("cars.idCar"))     
    car_owner = relationship("Car", back_populates="registers1")
    
    idTRReg = Column(Integer, ForeignKey("typeRegistersCar.idTR"))    
    type_reg_car_owner = relationship("TypeRegisterCar", back_populates="registers2")
    
    idPlaceOrigin = Column(Integer, ForeignKey("places.idPlace"))        
    place_owner = relationship("Place", back_populates="registers3", foreign_keys=[idPlaceOrigin])
    
    idPlaceDestination = Column(Integer, ForeignKey("places.idPlace"))    
    place_owner2 = relationship("Place", back_populates="registers4", foreign_keys=[idPlaceDestination])
    
    idUserReg = Column(Integer, ForeignKey("users.idUser"))     
    user_owner = relationship("User", back_populates="registers5")

```

Como se puede apreciar en el codigo, cada FK tiene un diferente `back_populates` y en la clase Place, cada `relationship` tiene su FK.

### 3.- Consideraciones

Estoy usando python 3.12 con un ambiente virtual pyevn:
 - SQLAlchemy 2.0.38
 - fastapi 0.110.0
 - pymysql 1.1.0