# SESION DE LABORATORIO N° 03: PATRONES DE DISEÑO DE COMPORTAMIENTO

## OBJETIVOS
  * Comprender el funcionamiento de algunos patrones de diseño de software del tipo de comportamiento.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de C# y Visual Studio Code.
  * Hardware:
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - Net 8 o superior
    - Visual Studio Code

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesarios

## DESARROLLO

### PARTE I: Strategy Design Pattern 

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/21cf440e-8156-498c-afd7-95c066ffaa93)
En la imagen Steve compra un monitor y una lavadora, pero a la hora de acercarse a la ventanilla existen tres formas de pagar: Tarjeta de Crédito, Tarjeta de Débito y Efectivo.

1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Payment
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Payment
dotnet new classlib -o Payment.Domain
dotnet sln add ./Payment.Domain/Payment.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new nunit -o Payment.Domain.Tests
dotnet sln add ./Payment.Domain.Tests/Payment.Domain.Tests.csproj
dotnet add ./Payment.Domain.Tests/Payment.Domain.Tests.csproj reference ./Payment.Domain/Payment.Domain.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Payment.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Payment.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. Primero se necesita implementar la interfaz que servirá de ESTRATEGIA base para las posibles implementaciones de pagos. Por eso en VS Code, en el proyecto Notifications.Domain proceder a crear el archivo IPaymentStrategy.cs :
```C#
namespace Payment.Domain
{
    public interface IPaymentStrategy
    {
        bool Pay(double amount);
    }
}
```
7. Ahora proceder a implementar las clases concretas o implementaciones a partir de la interfaz creada, Para esto en el proyecto Payment.Domain proceder a crear los archivos siguientes:
> CreditCardPaymentStrategy.cs
```C#
namespace Payment.Domain
{
    public class CreditCardPaymentStrategy : IPaymentStrategy
    {
        public bool Pay(double amount)
        {
            Console.WriteLine("Customer pays Rs " + amount + " using Credit Card");
            return true;
        }
    }
}
```
> DebitCardPaymentStrategy.cs
```C#
namespace Payment.Domain
{
    public class DebitCardPaymentStrategy : IPaymentStrategy
    {
        public bool Pay(double amount)
        {
            Console.WriteLine("Customer pays Rs " + amount + " using Debit Card");
            return true;
        }
    }
}
```
> CashPaymentStrategy.cs
```C#
namespace Payment.Domain
{
    public class CashPaymentStrategy : IPaymentStrategy
    {
        public bool Pay(double amount)
        {
            Console.WriteLine("Customer pays Rs " + amount + " By Cash");
            return true;
        }
    }
}
```
8. Seguidamente crear la clase que funcionara de contexto y permitira la ejecución de cualquier estrategia, por lo que en el proyecto de Payment.Domain se debe agregar el archivo PaymentContext.cs con el siguiente código:
```C#
namespace Payment.Domain
{
    public class PaymentContext
    {
        // The Context has a reference to the Strategy object.
        // The Context does not know the concrete class of a strategy. 
        // It should work with all strategies via the Strategy interface.
        private IPaymentStrategy PaymentStrategy;
        // The Client will set what PaymentStrategy to use by calling this method at runtime
        public void SetPaymentStrategy(IPaymentStrategy strategy)
        {
            PaymentStrategy = strategy;
        }
        // The Context delegates the work to the Strategy object instead of
        // implementing multiple versions of the algorithm on its own.
        public bool Pay(double amount)
        {
            return PaymentStrategy.Pay(amount);
        }
    }
}
```
9. Adicionalmente para facilitar la utilización de las diferentes estrategias adicionaremos una fachada, para eso crear el archivo PaymentService.cs en el proyecto Payment.Domain:
```C#
namespace Payment.Domain
{
    public class PaymentService
    {
        public bool ProcessPayment(int SelectedPaymentType, double Amount)
        {
            //Create an Instance of the PaymentContext class
            PaymentContext context = new PaymentContext();
            if (SelectedPaymentType == (int)PaymentType.CreditCard)
            {
                context.SetPaymentStrategy(new CreditCardPaymentStrategy());
            }
            else if (SelectedPaymentType == (int)PaymentType.DebitCard)
            {
                context.SetPaymentStrategy(new DebitCardPaymentStrategy());
            }
            else if (SelectedPaymentType == (int)PaymentType.Cash)
            {
                context.SetPaymentStrategy(new CashPaymentStrategy());
            }
            else
            {
                throw new ArgumentException("You Select an Invalid Payment Option");
            }
            //Finally, call the Pay Method
            return context.Pay(Amount);;
        }
    }
    public enum PaymentType
    {
        CreditCard = 1,  // 1 for CreditCard
        DebitCard = 2,   // 2 for DebitCard
        Cash = 3, // 3 for Cash
    }
}
```
10. Ahora proceder a implementar unas pruebas para verificar el correcto funcionamiento de la aplicación. Para esto al proyecto Payment.Domain.Tests adicionar el archivo PaymentTests.cs y agregar el siguiente código:
```C#
using System;
using NUnit.Framework;
using Payment.Domain;
namespace Payment.Domain.Tests
{
    public class PaymentTests
    {
        [TestCase(1, 1000)]
        [TestCase(2, 2000)]
        [TestCase(3, 3000)]
        public void GivenAValidPaymentTypeAndAmount_WhenProcessPayment_ResultIsSuccesful(int paymentType, double amount)
        {
            bool PaymentResult = new PaymentService().ProcessPayment(paymentType, amount);
            Assert.IsTrue(PaymentResult);
        }
        [TestCase(4, 4000)]
        public void GivenAnUnknownPaymentTypeAndAmount_WhenProcessPayment_ResultIsError(int paymentType, double amount)
        {
            //bool PaymentResult = new PaymentService().ProcessPayment(paymentType, amount);
            var ex = Assert.Throws<ArgumentException>(
                () => new PaymentService().ProcessPayment(paymentType, amount));
            Assert.That(ex.Message, Is.EqualTo("You Select an Invalid Payment Option"));
        }   
    }
}
```
11. Ahora necesitamos comprobar las pruebas contruidas para eso abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar los comandos:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
12. Si las pruebas se ejecutaron correctamente debera aparcer un resultado similar al siguiente:
```Bash
Passed!  - Failed:     0, Passed:     4, Skipped:     0, Total:     4, Duration: 12 ms
```

13. Finalmente se puede apreciar que existen tres componentes principales en el patrón ESTARTEGIA:
a. Estrategia: declarada en una interfac para ser implementada para todos los algoritmos soportado
b. EstrategiaConcreta: Es la implementa la estrategia para cada algoritmo
c. Conexto: esta es la clase que mantiene la referencia al objeto Estrategia y luego utiliza la referencia para llamar al algoritmo definido por cada EstrtaegiaConcreta

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/e132b3dd-1b5d-4cdf-a91d-fe114071c4bb)

14. En el terminal, ejecutar el siguiente comando para generar el diagrama de clases respectivo, tener en consideración que ruta del DLL puede ser distinta según la versión de .NET tenga instalada en el equipo.
```Bash
dotnet tool install --global dll2mmd
dll2mmd -f Payment.Domain/bin/Debug/net7.0/Payment.Domain.dll -o payment.md
```


### PARTE II: Command Design Pattern

1. Iniciar una nueva instancia de la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o ATM
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd ATM
dotnet new classlib -o ATM.Domain
dotnet sln add ./ATM.Domain/ATM.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new nunit -o ATM.Domain.Tests
dotnet sln add ./ATM.Domain.Tests/ATM.Domain.Tests.csproj
dotnet add ./ATM.Domain.Tests/ATM.Domain.Tests.csproj reference ./ATM.Domain/ATM.Domain.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto ATM.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto ATM.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. Inicialmente se necesita implementar la clase Cuenta que se utilizara en todas los comandos del ATM. Para esto crear el archivo Account.cs en el proyecto ATM.Domain con el siguiente código:
```C#
using System;
namespace ATM.Domain
{
    public class Account
    {
        public const decimal MAX_INPUT_AMOUNT = 10000;
        public int AccountNumber { get; set; }
        public decimal AccountBalance { get; set; }

        public void Withdraw(decimal amount)
        {
            if (amount > AccountBalance) 
                throw new ArgumentException("The input amount is greater than balance.");
            AccountBalance -= amount;            
        }
        public void Deposit(decimal amount)
        {
            if (amount > MAX_INPUT_AMOUNT) 
                throw new ArgumentException("The input amount is greater than maximum allowed.");
            AccountBalance += amount;            
        }
    }
}
```
7. Seguidamente se necesita implementar la interfaz principal para la generación de comandos, para esto crear el archivo ICommand.cs en el proyecto ATM.Domain con el siguiente código:
```C#
namespace ATM.Domain
{
    // Command Interface
    // It declares a method for executing a command
    public interface ICommand
    {
        void Execute();
    }
}
```
8. Ahora se debe implementar cada una de clases correspondiente a los comandos de Retirar y Depositar para eso se deberan crear los siguientes archivos con el còdigo correspondiente:
> WithdrawCommand.cs
```C#
namespace ATM.Domain
{
    public class WithdrawCommand : ICommand
    {
        Account _account;
        decimal _amount;
        public WithdrawCommand(Account account, decimal amount)
        {
            _account = account;
            _amount = amount;
        }
        public void Execute()
        {
            _account.Withdraw(_amount);
        }
    }
}
```
> DepositCommand.cs
```C#
namespace ATM.Domain
{
    public class DepositCommand : ICommand
    {
        Account _account;
        decimal _amount;
        public DepositCommand(Account account, decimal amount)
        {
            _account = account;
            _amount = amount;
        }
        public void Execute()
        {
            _account.Deposit(_amount);
        }        
    }
}
```

8. Finalmente para unir todos los comandos crear la clase ATM que permitira el manejo de los comandos, crear el archivo ATM.cs en el proyecto ATM.Domain:
```C#
namespace ATM.Domain
{
    public class ATM
    {
        ICommand _command;
        public ATM(ICommand command)
        {
            _command = command;
        }
        public void Action()
        {
            _command.Execute();
        }
    }
}
```

9. Para probar esta implementación, crear el archivo ATMTests.cs en el proyecto ATM.Domain.Tests:
```C#
using NUnit.Framework;
namespace ATM.Domain.Tests
{
    public class ATMTests
    {
        [Test]
        public void GivenAccountAndWithdraw_ThenExecute_ReturnsCorrectAmount()
        {
            var account = new Account() { AccountBalance = 300 };
            decimal amount = 100;
            var withdraw = new WithdrawCommand(account, amount);
            new ATM(withdraw).Action();
            Assert.IsTrue(account.AccountBalance.Equals(200));
        }
        [Test]
        public void GivenAccountAndDeposit_ThenExecute_ReturnsCorrectAmount()
        {
            var account = new Account() { AccountBalance = 200 };
            decimal amount = 100;
            var deposit = new DepositCommand(account, amount);
            new ATM(deposit).Action();
            Assert.IsTrue(account.AccountBalance.Equals(300));
        }
    }
}
```
10. Ahora necesitamos comprobar las pruebas contruidas para eso abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar el comando:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
11. Si las pruebas se ejecutaron correctamente debera aparcer un resultado similar al siguiente:
```Bash
Correctas! - Con error:     0, Superado:     2, Omitido:     0, Total:     2, Duración: 5 ms
```
11. Revisemos como funciona el patrón de diseño Comando.

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/50ecff5e-dc02-4b54-980f-8b72546b4129)

Como se puede apreciar la imagen, el patron de diseño Comando consiste de 5 componentes:

Receiver: Es la clase que contiene la actual implementacionde los metods que el cliente quiere invocar. En este ejemplo la cuenta.
Command: Esta viene a ser la interfaz que espeficica la operacion Ejecutar.
ConcreteCommand: Con las clases que implementa la interfaz ICommand y proporcionan las implementaciones del metodo Ejecutar. 
Invoker: El Invocador viene a ser la clase que resuelve que Command realiza determinada acción. En este caso la clase ATM.
Client: Es la clase que crea y ejecuta el comando.

12. En el terminal, ejecutar el siguiente comando para generar el diagrama de clases respectivo, tener en consideración que ruta del DLL puede ser distinta según la versión de .NET tenga instalada en el equipo.
```Bash
dotnet tool install --global dll2mmd
dll2mmd -f ATM.Domain/bin/Debug/net7.0/ATM.Domain.dll -o atm.md
```

---
## Actividades Encargadas
1. Completar la documentación de todas las clases y generar una automatización .github/workflows/publish_docs.yml (Github Workflow) utilizando DocFx (init, metadata y build) y publicar el site de documentación generado en un Github Page.
2. Generar una automatización de nombre .github/workflows/package_nuget.yml (Github Workflow) que ejecute:
   * Pruebas unitarias y reporte de pruebas automatizadas
   * Realice el analisis con SonarCloud.
   * Contruya un archivo .nuget a partir del proyecto Payment.Domain y del proyecto ATM.Domain y los publique como un Paquete de Github
3. Generar una automatización de nombre .github/workflows/release_version.yml (Github Workflow) que contruya la version (release) de cada paquete y publique en Github Releases e incluya los nugets generados

1. Crear un nuevo proyecto ```dotnet new sln -o Comportamiento``` el cual debe incluir su proyecto de dominio y su respectivo proyecto de pruebas utilizando otro patrón de diseño de COMPORTAMIENTO.
2. Crear un nuevo archivo Markdown llamado comportamiento.md que incluya el paso a paso del punto 1 incluyendo su diagrama generado en código Mermaid.
