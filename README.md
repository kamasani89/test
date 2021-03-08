/*
Enhance the Inventory class to have a add/remove/size/countByType method and passes the tests.
    - Data Types and Modifiers must stay unchanged
    - The inventory must group parts by type
    - The inventory must not add parts with existing name (ex. if part "a" exists, should not add any more parts with the same name)
    - Enhance TableParts and ChairParts getDescription() method to pass their tests
 */
import java.util.HashMap;
import java.util.List;
import java.util.*;
import java.util.stream.*;


enum PartsType {
    CHAIR_TYPE, TABLE_TYPE
}

class Inventory implements Storage {
    HashMap<PartsType, List<Part>> storage;

    Inventory() {
        this.storage = new HashMap<>();
    }

    @Override
    public void add(Part part) {
        PartsType partsType = part.getType();
        if(storage.containsKey(partsType)){
            List<Part> parts = storage.get(partsType);
            if(!parts.stream().anyMatch(a->a.name.equals(part.name))){
              parts.add(part);
              storage.put(partsType,parts);
            }
        }else{
          List<Part> parts = new ArrayList<>();
          parts.add(part);
          storage.put(partsType,parts);
        }
    }

    @Override
    public void remove(PartsType type, String partName) {
        
        if(storage.containsKey(type)){
            List<Part> parts = storage.get(type).stream().filter(a->!a.name.equals(partName)).collect(Collectors.toList());
            storage.put(type,parts);
        }

    }

    @Override
    public int size() {
        return storage.size();
    }

    @Override
    public int countByType(PartsType type) {
      
        if(storage.containsKey(type)){
          return storage.get(type).size();
        }
        return 0;
    }

    @Override
    public Integer countByType(Class<? extends Part> part) {

       if(part==ChairParts.class){
          return storage.get(PartsType.CHAIR_TYPE).size();
       }else if(part==TableParts.class){
         return storage.get(PartsType.TABLE_TYPE).size();
       }
       return 0;
    }

    @Override
    public void remove(Class<? extends Part> part, String partName) {
        if( part == ChairParts.class){
           remove(PartsType.CHAIR_TYPE,partName);
        }else{
          remove(PartsType.TABLE_TYPE,partName);
        }
    }

    @Override
    public boolean containsName(List<Part> list, String name) {
        return list.stream().anyMatch(a->a.name.equals(name));
    }
}

interface Storage {
    void add(Part part);

    void remove(PartsType type, String partName);

    int size();

    int countByType(PartsType type);

    Integer countByType(Class<? extends Part> part);

    void remove(Class<? extends Part> part, String partName);

    boolean containsName(final List<Part> list, final String name);
}

abstract class Part {
    protected PartsType type;
    protected String name;

    Part(PartsType type, String name) {
        this.name = name;
        this.type = type;
    }

    public PartsType getType() {
        return type;
    }

    public String getDescription() {
        return "";
    }
}

class ChairParts extends Part {

    ChairParts(String name) {
        super(PartsType.CHAIR_TYPE, name);
    }

    @Override
    public PartsType getType() {
        return super.type;
    }

    @Override
    public String getDescription() {
        return "(" + this.name + ")";
    }
}

class TableParts extends Part {

    TableParts(String name) {
        super(PartsType.TABLE_TYPE, name);
    }

    @Override
    public PartsType getType() {
        return super.type;
    }

    @Override
    public String getDescription() {
        return "((" + this.name + "))";
    }


}

class Factory {
    Part createTable(String name) {
        return new TableParts (name);
    }

    Part createChair(String name) {
        return new ChairParts (name);
    }
}
class Main {
    public static void main(String[] args) {
        run(new Inventory(), new Factory());
    }

    public static void run(Inventory storage, Factory factory) {

        storage.add(factory.createTable("a"));

        storage.add(factory.createChair("a"));
        storage.add(factory.createChair("b"));
        storage.add(factory.createChair("b"));
        storage.add(factory.createChair("c"));

        assertEqual(storage.size(), 2);
        assertEqual(storage.countByType(PartsType.TABLE_TYPE), 1);
        assertEqual(storage.countByType(PartsType.CHAIR_TYPE), 3);

        storage.remove(PartsType.TABLE_TYPE, "a");
        storage.remove(PartsType.TABLE_TYPE, "a");
        assertEqual(storage.countByType(PartsType.TABLE_TYPE), 0);

        Part newChair = factory.createChair("newChair");
        Part newTable = factory.createTable("newTable");

        assertEqual(newChair.getDescription(), "(newChair)");
        assertEqual(newTable.getDescription(), "((newTable))");

        storage.add(newChair);
        assertEqual(storage.countByType(PartsType.CHAIR_TYPE), 4);
        storage.remove(ChairParts.class, "newChair");
        assertEqual(storage.countByType(ChairParts.class), 3); 


        System.out.println("done.");

    }

    static void assertEqual(Object o1, Object o2) {
        if (o1 == o2 || o1.equals(o2)) {
            return;
        }
        throw new RuntimeException("Assertion failed " + o1.toString() + " is not equal to " + o2.toString());
    }

}
