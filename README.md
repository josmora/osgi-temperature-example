# OSGi Temperature Sensor Example
This repository contains an example of using OSGi R6 Declarative Services (DS) to demonstrate dynamic service binding and filtering in a modular Java application. The example includes a TemperatureSensor interface, two different implementations of the interface by different manufacturers ("Acme Inc. TS1000" and "Widget Corp. WX2000"), and a TemperatureMonitor component that consumes the TemperatureSensor services based on specific conditions.

The example showcases the following concepts:

Registering and consuming services using OSGi R6 annotations.
Filtering services based on custom properties (manufacturer and model number).
Dynamically selecting and binding to services at runtime based on specific conditions.
This example is suitable for developers who want to learn how to create modular Java applications using OSGi, particularly focusing on the dynamic and flexible nature of OSGi service binding and filtering.


##OSGI
 OSGi (Open Services Gateway initiative) is a modular and dynamic Java framework that enables the creation, deployment, and management of software components, known as bundles, in a runtime environment. One key feature of OSGi is its service registry, which allows bundles to discover and interact with services provided by other bundles. OSGi service filters play a crucial role in this process.

OSGi service filters are expressions used to narrow down the search for services based on specific criteria. They follow the LDAP (Lightweight Directory Access Protocol) filter syntax and enable bundles to find and bind to services that match the specified criteria. This approach enhances the flexibility and dynamic nature of OSGi, allowing bundles to adapt to changes in available services at runtime.

Here's how OSGi service filters work:

Bundle registration: When a bundle registers a service, it may provide a set of properties as a dictionary or a map. These properties describe the service and can be used by other bundles to identify and filter the services they want to use.

Filter creation: A bundle that wants to find a specific service creates a service filter using LDAP filter syntax. The filter can specify criteria based on the service's properties, such as service interface, version, vendor, or other custom properties.

Service lookup: The bundle passes the filter to the OSGi framework when looking up a service. The framework then searches the service registry and returns a list of services that match the filter criteria.

Service binding: The bundle can then bind to one or more of the services returned by the lookup. If multiple services match the filter, the bundle can choose one based on other factors like service ranking or service ID.

Dynamic adaptation: When a service's properties change, or a new service is registered that matches the filter, the OSGi framework will notify the bundle. The bundle can then update its bindings to use the new or modified service.

In summary, OSGi service filters provide a powerful and dynamic way to discover and interact with services in an OSGi environment. By specifying criteria based on service properties, bundles can efficiently find and bind to the services they need, and adapt to changes in available services at runtime.

## Example
Imagine we have two services that implement the same interface, `TemperatureSensor`, but they are produced by different manufacturers and have different model numbers. In this example, a bundle is interested in using a temperature sensor, but it specifically wants to use one from the manufacturer "Acme Inc." and with the model number "TS1000". 

Here's how this scenario would play out in an OSGi environment:

1. Service registration: Both temperature sensor services are registered with the OSGi service registry by their respective bundles, along with their properties:

TemperatureSensor Service 1:
- Interface: TemperatureSensor
- Manufacturer: Acme Inc.
- Model: TS1000

TemperatureSensor Service 2:
- Interface: TemperatureSensor
- Manufacturer: Widget Corp.
- Model: WX2000

2. Filter creation: The bundle looking for a specific temperature sensor service creates a filter using LDAP filter syntax, specifying the desired interface, manufacturer, and model number:

```
(&(objectClass=TemperatureSensor)(manufacturer=Acme Inc.)(model=TS1000))
```

3. Service lookup: The bundle performs a service lookup with the filter, and the OSGi framework returns a list of matching services. In this case, only the "Acme Inc. TS1000" service matches the filter criteria.

4. Service binding: The bundle binds to the returned "Acme Inc. TS1000" service and starts using it.

By using a service filter, the bundle was able to find and use the specific temperature sensor service it wanted, even though there were multiple services implementing the same interface. The filter enabled the bundle to choose the desired service based on custom properties like manufacturer and model number.

## Using R6 annotations
In this example, we'll demonstrate how to use OSGi R6 annotations and the OSGi Declarative Services (DS) to register and consume the `TemperatureSensor` services with specific properties. We'll create two temperature sensor implementations, one for "Acme Inc." and another for "Widget Corp.", and a `TemperatureMonitor` that consumes the "Acme Inc. TS1000" service using a filter.

1. Define the `TemperatureSensor` interface:

```java
public interface TemperatureSensor {
    double readTemperature();
    String getManufacturer();
    String getModel();
}
```

2. Create the "Acme Inc. TS1000" `TemperatureSensor` implementation:

```java
import org.osgi.service.component.annotations.Component;

@Component(property = {"manufacturer=Acme Inc.", "model=TS1000"})
public class AcmeTS1000TemperatureSensor implements TemperatureSensor {
    @Override
    public double readTemperature() {
        return 25.0; // Replace this with the actual temperature reading logic
    }

    @Override
    public String getManufacturer() {
        return "Acme Inc.";
    }

    @Override
    public String getModel() {
        return "TS1000";
    }
}
```

3. Create the "Widget Corp. WX2000" `TemperatureSensor` implementation:

```java
import org.osgi.service.component.annotations.Component;

@Component(property = {"manufacturer=Widget Corp.", "model=WX2000"})
public class WidgetWX2000TemperatureSensor implements TemperatureSensor {
    @Override
    public double readTemperature() {
        return 25.5; // Replace this with the actual temperature reading logic
    }

    @Override
    public String getManufacturer() {
        return "Widget Corp.";
    }

    @Override
    public String getModel() {
        return "WX2000";
    }
}
```

4. Create the `TemperatureMonitor` component that consumes the "Acme Inc. TS1000" `TemperatureSensor` service:

```java
import org.osgi.service.component.annotations.Activate;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

@Component
public class TemperatureMonitor {
    private TemperatureSensor temperatureSensor;

    @Reference(target = "(&(manufacturer=Acme Inc.)(model=TS1000))")
    public void setTemperatureSensor(TemperatureSensor temperatureSensor) {
        this.temperatureSensor = temperatureSensor;
    }

    public void unsetTemperatureSensor(TemperatureSensor temperatureSensor) {
        this.temperatureSensor = null;
    }

    @Activate
    public void activate() {
        System.out.println("TemperatureMonitor activated");
        System.out.println("Using temperature sensor: " + temperatureSensor.getManufacturer() + " " + temperatureSensor.getModel());
        System.out.println("Current temperature: " + temperatureSensor.readTemperature());
    }
}
```

In this example, the `TemperatureMonitor` component uses the `@Reference` annotation with a `target` filter to consume the "Acme Inc. TS1000" `TemperatureSensor` service. The OSGi framework will automatically bind the matching `TemperatureSensor` service to the `TemperatureMonitor` component at runtime.


## Choosing Service at runtime 
In this example, we'll create a `TemperatureMonitor` component that dynamically selects a `TemperatureSensor` service at runtime based on a specific condition. Let's say the condition is the lowest temperature reading from the available services. We'll use OSGi Declarative Services (DS) to handle the service binding and selection.

1. Define the `TemperatureSensor` interface:

```java
public interface TemperatureSensor {
    double readTemperature();
    String getManufacturer();
    String getModel();
}
```

2. Create the "Acme Inc. TS1000" and "Widget Corp. WX2000" `TemperatureSensor` implementations as shown in the previous example.

3. Create the `TemperatureMonitor` component that dynamically selects a `TemperatureSensor` service at runtime:

```java
import org.osgi.service.component.annotations.Activate;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Deactivate;
import org.osgi.service.component.annotations.Reference;
import org.osgi.service.component.annotations.ReferenceCardinality;
import org.osgi.service.component.annotations.ReferencePolicy;

import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

@Component(immediate = true)
public class TemperatureMonitor {
    private final ConcurrentMap<String, TemperatureSensor> temperatureSensors = new ConcurrentHashMap<>();
    private TemperatureSensor selectedSensor;

    @Reference(cardinality = ReferenceCardinality.MULTIPLE, policy = ReferencePolicy.DYNAMIC)
    public void addTemperatureSensor(TemperatureSensor temperatureSensor) {
        String key = temperatureSensor.getManufacturer() + " " + temperatureSensor.getModel();
        temperatureSensors.put(key, temperatureSensor);
        updateSelectedSensor();
    }

    public void removeTemperatureSensor(TemperatureSensor temperatureSensor) {
        String key = temperatureSensor.getManufacturer() + " " + temperatureSensor.getModel();
        temperatureSensors.remove(key);
        updateSelectedSensor();
    }

    private void updateSelectedSensor() {
        TemperatureSensor lowestTemperatureSensor = null;
        double lowestTemperature = Double.MAX_VALUE;

        for (TemperatureSensor sensor : temperatureSensors.values()) {
            double temperature = sensor.readTemperature();
            if (temperature < lowestTemperature) {
                lowestTemperature = temperature;
                lowestTemperatureSensor = sensor;
            }
        }

        selectedSensor = lowestTemperatureSensor;
    }

    @Activate
    public void activate() {
        System.out.println("TemperatureMonitor activated");
    }

    @Deactivate
    public void deactivate() {
        System.out.println("TemperatureMonitor deactivated");
    }

    public void printSelectedSensor() {
        if (selectedSensor != null) {
            System.out.println("Selected temperature sensor: " + selectedSensor.getManufacturer() + " " + selectedSensor.getModel());
            System.out.println("Current temperature: " + selectedSensor.readTemperature());
        } else {
            System.out.println("No temperature sensor available");
        }
    }
}
```

In this example, the `TemperatureMonitor` component uses the `@Reference` annotation with the `MULTIPLE` cardinality and `DYNAMIC` policy to manage a collection of `TemperatureSensor` services. The `addTemperatureSensor` and `removeTemperatureSensor` methods are called by the OSGi framework when a `TemperatureSensor` service is registered or unregistered. The `updateSelectedSensor` method dynamically selects the sensor with the lowest temperature reading.

To select a service at runtime, you could call the `printSelectedSensor` method, which will print the currently selected `TemperatureSensor` based on the lowest temperature criterion.