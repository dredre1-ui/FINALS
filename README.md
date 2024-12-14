# FINALS
using PdfSharp.Drawing;
using PdfSharp.Pdf;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

class Item
{
    public string Name { get; set; }
    public int Quantity { get; set; }
    public decimal Price { get; set; }

    public override string ToString()
    {
        return $"{Name}|{Quantity}|{Price}";
    }

    public static Item FromString(string itemString)
    {
        var parts = itemString.Split('|');
        return new Item
        {
            Name = parts[0],
            Quantity = int.Parse(parts[1]),
            Price = decimal.Parse(parts[2])
        };
    }
}

class Inventory
{
    private List<Item> items = new List<Item>();
    private const string FilePath = "product_file.txt";

    public Inventory()
    {
        LoadFromFile();
    }

    public void AddItem(Item item)
    {
        items.Add(item);
        SaveToFile();
    }

    public void RemoveItem(string name)
    {
        items.RemoveAll(i => i.Name.Equals(name, StringComparison.OrdinalIgnoreCase));
        SaveToFile();
    }

    public void UpdateItem(string name, int newQuantity, decimal newPrice)
    {
        var item = items.Find(i => i.Name.Equals(name, StringComparison.OrdinalIgnoreCase));
        if (item != null)
        {
            item.Quantity = newQuantity;
            item.Price = newPrice;
            SaveToFile();
        }
    }

    public void DisplayInventory()
    {
        Console.WriteLine("====================================");
        Console.WriteLine("Coffee Shop Inventory");
        Console.WriteLine("====================================");

        Console.WriteLine($"{"Name",-20} {"Quantity",10} {"Price (PHP)",15}");
        Console.WriteLine(new string('-', 45));

        foreach (var item in items)
        {
            Console.WriteLine($"{item.Name,-20} {item.Quantity,10} {item.Price,15:0.00}");
        }
    }

    public List<Item> GetItems() => items;

    public void SearchItem(string itemName)
    {
        itemName = itemName?.Trim();  

        if (string.IsNullOrEmpty(itemName))
        {
            Console.WriteLine("Please enter a valid item name to search.");
            return;
        }

        
        Console.WriteLine($"Searching for: '{itemName}'");

        var foundItems = items.FindAll(i => i.Name.Contains(itemName, StringComparison.OrdinalIgnoreCase));

        if (foundItems.Count > 0)
        {
            Console.WriteLine("Search Results:");
            Console.WriteLine($"{"Name",-20} {"Quantity",10} {"Price (PHP)",15}");
            Console.WriteLine(new string('-', 45));

            foreach (var item in foundItems)
            {
                Console.WriteLine($"{item.Name,-20} {item.Quantity,10} {item.Price,15:0.00}");
            }
        }
        else
        {
            Console.WriteLine($"No items found matching '{itemName}'. Please try again with a different search term.");
        }
    }

    private void SaveToFile()
    {
        File.WriteAllLines(FilePath, items.Select(i => i.ToString()));
    }

    private void LoadFromFile()
    {
        if (File.Exists(FilePath))
        {
            var lines = File.ReadAllLines(FilePath);
            items = lines.Select(line => Item.FromString(line)).ToList();
        }
    }

    public void GenerateInventoryReport(string outputPath)
    {
        var document = new PdfDocument();
        document.Info.Title = "Inventory Report";

        var page = document.AddPage();
        var gfx = XGraphics.FromPdfPage(page);
        var font = new XFont("Verdana", 12);
        var headerFont = new XFont("Verdana", 14);
        var headerColor = XColor.FromArgb(0, 0, 0); 

        int yPoint = 40;

        gfx.DrawString("Coffee Shop Inventory Report", new XFont("Verdana", 16), XBrushes.Black, new XPoint(40, 20));

        gfx.DrawString("Name", headerFont, new XSolidBrush(headerColor), new XPoint(40, yPoint));
        gfx.DrawString("Quantity", headerFont, new XSolidBrush(headerColor), new XPoint(200, yPoint));
        gfx.DrawString("Price (PHP)", headerFont, new XSolidBrush(headerColor), new XPoint(300, yPoint));

        yPoint += 20;

        foreach (var item in items)
        {
            gfx.DrawString(item.Name, font, XBrushes.Black, new XPoint(40, yPoint));
            gfx.DrawString(item.Quantity.ToString(), font, XBrushes.Black, new XPoint(200, yPoint));
            gfx.DrawString(item.Price.ToString("0.00"), font, XBrushes.Black, new XPoint(300, yPoint));
            yPoint += 20;
        }

        document.Save(outputPath);
        Console.WriteLine($"Inventory report saved to {outputPath}");
    }
}

class SalesReport
{
    private List<(DateTime Date, List<(string Name, int Quantity, decimal Price)>)> sales = new List<(DateTime, List<(string, int, decimal)>)>();

    public void AddSale(List<(string Name, int Quantity, decimal Price)> items)
    {
        sales.Add((DateTime.Now, items));
    }

    public void GenerateSalesReport(string outputPath)
    {
        var document = new PdfDocument();
        document.Info.Title = "Sales Report";

        var page = document.AddPage();
        var gfx = XGraphics.FromPdfPage(page);
        var font = new XFont("Verdana", 12);
        var headerFont = new XFont("Verdana", 14);
        var headerColor = XColor.FromArgb(0, 0, 0);

        int yPoint = 40;

        gfx.DrawString("Coffee Shop Sales Report", new XFont("Verdana", 16), XBrushes.Black, new XPoint(40, 20));

        gfx.DrawString("Date", headerFont, new XSolidBrush(headerColor), new XPoint(40, yPoint));
        gfx.DrawString("Name", headerFont, new XSolidBrush(headerColor), new XPoint(150, yPoint));
        gfx.DrawString("Quantity", headerFont, new XSolidBrush(headerColor), new XPoint(300, yPoint));
        gfx.DrawString("Price (PHP)", headerFont, new XSolidBrush(headerColor), new XPoint(400, yPoint));
        yPoint += 20;

        decimal totalSales = 0;

        foreach (var sale in sales)
        {
            gfx.DrawString(sale.Date.ToString("yyyy-MM-dd HH:mm:ss"), font, XBrushes.Black, new XPoint(40, yPoint));
            yPoint += 20;

            foreach (var item in sale.Item2)
            {
                gfx.DrawString(item.Name, font, XBrushes.Black, new XPoint(150, yPoint));
                gfx.DrawString(item.Quantity.ToString(), font, XBrushes.Black, new XPoint(300, yPoint));
                gfx.DrawString((item.Price * item.Quantity).ToString("0.00"), font, XBrushes.Black, new XPoint(400, yPoint));
                totalSales += item.Price * item.Quantity;
                yPoint += 20;
            }
        }

        gfx.DrawString($"Total Sales: PHP {totalSales:0.00}", font, XBrushes.Black, new XPoint(40, yPoint + 20));
        document.Save(outputPath);
        Console.WriteLine($"Sales report saved to {outputPath}");
    }
}

class Order
{
    public List<(string Name, int Quantity, decimal Price)> Items { get; set; } = new List<(string, int, decimal)>();
    public decimal TotalPrice { get; set; }

    private SalesReport salesReport;

    public Order(SalesReport salesReport)
    {
        this.salesReport = salesReport;
    }

    public void AddItem(Inventory inventory, string itemName, int quantity)
    {
        var item = inventory.GetItems().Find(i => i.Name.Equals(itemName, StringComparison.OrdinalIgnoreCase));
        if (item != null && item.Quantity >= quantity)
        {
            Items.Add((item.Name, quantity, item.Price));
            TotalPrice += item.Price * quantity;
            inventory.UpdateItem(item.Name, item.Quantity - quantity, item.Price);
        }
        else
        {
            Console.WriteLine("Item not found in inventory or insufficient quantity.");
        }
    }

    public void GenerateOrderSummary(string outputPath)
    {
        var document = new PdfDocument();
        document.Info.Title = "Order Summary";

        var page = document.AddPage();
        var gfx = XGraphics.FromPdfPage(page);
        var font = new XFont("Verdana", 12);
        var headerFont = new XFont("Verdana", 14);
        var headerColor = XColor.FromArgb(0, 0, 0);

        int yPoint = 40;

        gfx.DrawString("Coffee Shop Order Summary", new XFont("Verdana", 16), XBrushes.Black, new XPoint(40, 20));

        gfx.DrawString("Name", headerFont, new XSolidBrush(headerColor), new XPoint(40, yPoint));
        gfx.DrawString("Quantity", headerFont, new XSolidBrush(headerColor), new XPoint(200, yPoint));
        gfx.DrawString("Price (PHP)", headerFont, new XSolidBrush(headerColor), new XPoint(300, yPoint));

        yPoint += 20;

        foreach (var item in Items)
        {
            gfx.DrawString(item.Name, font, XBrushes.Black, new XPoint(40, yPoint));
            gfx.DrawString(item.Quantity.ToString(), font, XBrushes.Black, new XPoint(200, yPoint));
            gfx.DrawString((item.Price * item.Quantity).ToString("0.00"), font, XBrushes.Black, new XPoint(300, yPoint));
            yPoint += 20;
        }

        gfx.DrawString($"Total Price: PHP {TotalPrice:0.00}", font, XBrushes.Black, new XPoint(40, yPoint + 20));
        document.Save(outputPath);
        Console.WriteLine($"Order summary saved to {outputPath}");
    }

    internal void ProcessOrder(Inventory inventory)
    {
        
        if (Items.Count == 0)
        {
            Console.WriteLine("The order is empty. Please add items to the order before processing.");
            return;
        }

      
        foreach (var orderItem in Items)
        {
            var item = inventory.GetItems().Find(i => i.Name.Equals(orderItem.Name, StringComparison.OrdinalIgnoreCase));
            if (item != null)
            {
                
                if (item.Quantity < orderItem.Quantity)
                {
                    Console.WriteLine($"Not enough stock for {orderItem.Name}. Only {item.Quantity} available.");
                }
                else
                {
                    
                    inventory.UpdateItem(orderItem.Name, item.Quantity - orderItem.Quantity, item.Price);
                }
            }
            else
            {
                Console.WriteLine($"Item {orderItem.Name} not found in inventory.");
            }
        }

      
        salesReport.AddSale(Items);

        
        Console.WriteLine("Order processed successfully.");
    }
}

class Program
{
    static void Main(string[] args)
    {
        Inventory inventory = new Inventory();
        SalesReport salesReport = new SalesReport();
        Order order = new Order(salesReport);

        while (true)
        {
            DisplayTitle("Main Menu");
            Console.WriteLine("1. Manage Inventory");
            Console.WriteLine("2. Place Order");
            Console.WriteLine("3. Generate Sales Report");
            Console.WriteLine("4. Exit");

            int choice = GetIntInput("Enter your choice: ");

            switch (choice)
            {
                case 1:
                    ManageInventory(inventory);
                    break;
                case 2:
                    PlaceOrder(inventory, order);
                    break;
                case 3:
                    salesReport.GenerateSalesReport("SalesReport.pdf");
                    break;
                case 4:
                    return;
                default:
                    Console.WriteLine("Invalid choice.");
                    break;
            }
        }
    }

    static void DisplayTitle(string title)
    {
        Console.Clear();
        Console.WriteLine($"=== {title} ===");
    }

    static int GetIntInput(string prompt)
    {
        int choice;
        while (true)
        {
            Console.Write(prompt);
            if (int.TryParse(Console.ReadLine(), out choice)) break;
            Console.WriteLine("Invalid input. Please enter a number.");
        }
        return choice;
    }

    static decimal GetDecimalInput(string prompt)
    {
        decimal price;
        while (true)
        {
            Console.Write(prompt);
            if (decimal.TryParse(Console.ReadLine(), out price)) break;
            Console.WriteLine("Invalid input. Please enter a valid number.");
        }
        return price;
    }

    static void ManageInventory(Inventory inventory)
    {
        while (true)
        {
            Console.Clear();
            inventory.DisplayInventory();
            Console.WriteLine("1. Add Item");
            Console.WriteLine("2. Remove Item");
            Console.WriteLine("3. Update Item");
            Console.WriteLine("4. Search Item");
            Console.WriteLine("5. Generate Inventory Report");
            Console.WriteLine("6. Back");

            int choice = GetIntInput("Enter your choice: ");

            switch (choice)
            {
                case 1:
                    AddItem(inventory);
                    break;
                case 2:
                    RemoveItem(inventory);
                    break;
                case 3:
                    UpdateItem(inventory);
                    break;
                case 4:
                    string searchTerm = GetStringInput("Enter item name to search: ");
                    inventory.SearchItem(searchTerm);
                    break;
                case 5:
                    inventory.GenerateInventoryReport("InventoryReport.pdf");
                    break;
                case 6:
                    return;
                default:
                    Console.WriteLine("Invalid choice.");
                    break;
            }
        }
    }

    static void AddItem(Inventory inventory)
    {
        string name = GetStringInput("Enter item name: ");
        int quantity = GetIntInput("Enter item quantity: ");
        decimal price = GetDecimalInput("Enter item price: ");
        inventory.AddItem(new Item { Name = name, Quantity = quantity, Price = price });
    }

    static void RemoveItem(Inventory inventory)
    {
        string name = GetStringInput("Enter item name to remove: ");
        inventory.RemoveItem(name);
    }

    static void UpdateItem(Inventory inventory)
    {
        string name = GetStringInput("Enter item name to update: ");
        int quantity = GetIntInput("Enter new quantity: ");
        decimal price = GetDecimalInput("Enter new price: ");
        inventory.UpdateItem(name, quantity, price);
    }

    static void SearchItem(Inventory inventory)
    {
        string name = GetStringInput("Enter item name to search: ");
        inventory.SearchItem(name);
    }

    static string GetStringInput(string prompt)
    {
        Console.Write(prompt);
        return Console.ReadLine();
    }

    static void PlaceOrder(Inventory inventory, Order order)
    {
        while (true)
        {
            Console.Clear();
            Console.WriteLine("=== Place Order ===");
            Console.WriteLine("Available Inventory:");
            inventory.DisplayInventory();

            Console.WriteLine("\nEnter items for the order (or type 'done' to finish):");

            string itemName = GetStringInput("Item name: ");
            if (itemName.ToLower() == "done") break;

            int quantity = GetIntInput("Quantity: ");
            order.AddItem(inventory, itemName, quantity);
        }

        order.ProcessOrder(inventory);
    }
}
