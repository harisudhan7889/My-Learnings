# Serialization

* Java platform allow us to create objects in memory but the life span of the object will be until the Java Virtual Machine remains running. To make the object to stay beyond the lifetime of the Virtual Memory we use Serialization. 

* Object serialization is the process of saving an object's state to a sequence of bytes, as well as the process of rebuilding those bytes into a live object at some future time.
* We also call these objects as a persistent objects.

**How to make a object persistent?**

If a class implements the Serializable then this class's object can be persisted.

**What is there in Serializable?** 

No methods available in the serializable. It is interface with no methods. These kind of interface is called as Marker Interface.  It simply allows the serialization mechanism to verify that the class is able to be persisted. 

**Code**

*Serialization*

```java
10  import java.io.ObjectOutputStream;
20  import java.io.FileOutputStream;
30  import java.io.IOException;
40  public class FlattenObject
50  {
60    public static void main(String [] args)
70    {
80      String filename = "persistentobject.ser";
90      if(args.length > 0)
100     {
110       filename = args[0];
120     } 
130     PersistentObject object = new PersistentObject();
140     FileOutputStream fos = null;
150     ObjectOutputStream out = null;
160     try
170     {
180       fos = new FileOutputStream(filename);
190       out = new ObjectOutputStream(fos);
200       out.writeObject(object);
210       out.close();
220     }
230     catch(IOException ex)
240     {
250       ex.printStackTrace();
260     }
270   }
280 }
```

*Deserialization*

```java
10  import java.io.ObjectInputStream;
20  import java.io.FileInputStream;
30  import java.io.IOException;
40  import java.util.Calendar;
50  public class InflateObject
60  {
70    public static void main(String [] args)
80    {
90      String filename = "persistentobject.ser"; 
100     if(args.length > 0)
110     {
120       filename = args[0];
130     }
140   PersistentObject time = null;
150   FileInputStream fis = null;
160   ObjectInputStream in = null;
170   try
180   {
190     fis = new FileInputStream(filename);
200     in = new ObjectInputStream(fis);
210     time = (PersistentObject)in.readObject();
220     in.close();
230   }
240   catch(IOException ex)
250   {
260     ex.printStackTrace();
270   }
280   catch(ClassNotFoundException ex)
290   {
300     ex.printStackTrace();
310   }
320 }
330}
```

* The `java.lang.Object` class does not implement that interface. Therefore, not all the objects in Java can be persisted automatically. `java.lang.Object` not implementing the `Serializable` interface because any class you create that extends only `Object`, So `java.lang.Object`is not serializable unless you implement the interface yourself.
* If you do not want a variable inside the serializable class to be serialized then you have to mention it as **transient** like below

```java
class PersistentObject implements Serializable {
    int value1;
    int value2;
    transient int value3;
}
```

* Remember, a constructor is called only when a new instance is created. We are not creating a new instance at the time deserialization, we are restoring a persisted object. So suppose if anything done inside the constructor will not be invoked at the time of restoration. For example

  ```java
  class PersistentObject implements Serializable {
      int value1;
      int value2;
      transient int value3;
      PersistentObject() {
          calculateValues();
      }
      
      private void calculateValues() {
          ...
      }
  }
  ```

  So in the above case *calculateValues* method will be called only once at the time of object initialization. But it should also be called at the time of restoration. There is, however, a strange yet crafty solution. By using a built-in feature of the serialization mechanism, developers can enhance the normal process by providing two methods inside their class files. Those methods are:

  ```Java
  private void writeObject(ObjectOutputStream out) throws IOException;
  private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException;
  ```

  Notice that both methods are (and must be) declared `private`, proving that neither method is inherited and overridden or overloaded. The trick here is that the virtual machine will automatically check to see if either method is declared during the corresponding method call. The virtual machine can call private methods of your class whenever it wants but no other objects can. Thus, the integrity of the class is maintained and the serialization protocol can continue to work as normal. The serialization protocol is always used the same way, by calling either `ObjectOutputStream.writeObject()` or `ObjectInputStream.readObject()`. So, even though those specialized private methods are provided, the object serialization works the same way as far as any calling object is concerned. 

  Now we can see the solution below

  ```java
  class PersistentObject implements Serializable {
      int value1;
      int value2;
      transient int value3;
      PersistentObject() {
          calculateValues();
      }
      
      private void writeObject(ObjectOutputStream out) throws IOException {
          out.defaultWriteObject(); 
      }
      
      private void readObject(ObjectInputStream in) throws IOException,     		
      ClassNotFoundException {
          in.defaultReadObject();
          calculateValues();
      }
      
      private void calculateValues() {
          ...
      }
  }
  ```

  Notice the first line of each of the new private methods. Those calls do what they sound like -- they perform the default writing and reading of the flattened object, which is important because we are not replacing the normal process, we are only adding to it. Those methods work because the call to `ObjectOutputStream.writeObject()` kicks off the serialization protocol.

  Those private methods can be used for any customization you need to make to the serialization process. Encryption could be added to the output and decryption to the input (note that the bytes are written and read in cleartext with no obfuscation at all). They could be used to add extra data to the stream, perhaps a company versioning code. The possibilities are truly limitless.

*  What if you create a class whose superclass is serializable but you do not want that new class to be serializable? You cannot unimplemented an interface, so if your superclass does implement `Serializable`, your new class implements it, too (assuming both rules listed above are met). To stop the automatic serialization, you can once again use the private methods to just throw the `NotSerializableException`. 

```Java
private void writeObject(ObjectOutputStream out) throws IOException {
        throw new NotSerializableException("Not today!");
    }
    
private void readObject(ObjectInputStream in) throws IOException,     		
    ClassNotFoundException {
        throw new NotSerializableException("Not today!");
    }
```

* You can also customize the write and read using Externalizable which contains two methods

  ```java
  public void writeExternal(ObjectOutput out) throws IOException;
  public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
  ```

  Just override those methods to provide your own protocol. Unlike the previous two serialization variations, nothing is provided for free here, though. That is, the protocol is entirely in your hands. Although it's the more difficult scenario, it's also the most controllable. An example situation for that alternate type of serialization: read and write PDF files with a Java application. If you know how to write and read PDF (the sequence of bytes required), you could provide the PDF-specific protocol in the `writeExternal` and `readExternal` methods.

