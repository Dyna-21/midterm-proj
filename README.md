using System; using System.Collections.Generic; using System.Linq;

namespace PersonalBudgetTracker
{
    public enum TransactionType { Income, Expense }

    public class Transaction
    {
        public string Description { get; set; }
        public decimal Amount { get; set; }
        public TransactionType Type { get; set; }
        public string Category { get; set; }
        public DateTime Date { get; set; }

        public Transaction(string description, decimal amount, TransactionType type, string category, DateTime date)
        {
            Description = description;
            Amount = amount;
            Type = type;
            Category = category;
            Date = date;
        }
    }

    public class BudgetTracker
    {
        private List<Transaction> transactions = new List<Transaction>();

        public void AddTransaction(Transaction transaction)
        {
            transactions.Add(transaction);
        }

        public decimal GetTotalIncome() => transactions.Where(t => t.Type == TransactionType.Income).Sum(t => t.Amount);

        public decimal GetTotalExpenses() => transactions.Where(t => t.Type == TransactionType.Expense).Sum(t => t.Amount);

        public decimal GetNetSavings() => GetTotalIncome() - GetTotalExpenses();

        public Dictionary<string, decimal> GetCategoryAnalytics()
        {
            return transactions
                .Where(t => t.Type == TransactionType.Expense)
                .GroupBy(t => t.Category)
                .ToDictionary(g => g.Key, g => g.Sum(t => t.Amount));
        }

        public List<Transaction> SortTransactionsByDate() => transactions.OrderBy(t => t.Date).ToList();

        public List<Transaction> SortTransactionsByCategory() => transactions.OrderBy(t => t.Category).ToList();

        public List<Transaction> SortTransactionsByAmount() => transactions.OrderByDescending(t => t.Amount).ToList();

        public void DisplayTextGraph()
        {
            var analytics = GetCategoryAnalytics();
            Console.WriteLine("\nCategory-wise Expense Chart:");
            foreach (var item in analytics)
            {
                Console.WriteLine($"{item.Key}: {new string('*', (int)(item.Value / 10))} ({item.Value:C})");
            }
        }

        public void DisplayInsights()
        {
            var analytics = GetCategoryAnalytics();
            if (analytics.Any())
            {
                var maxCategory = analytics.Aggregate((l, r) => l.Value > r.Value ? l : r);
                Console.WriteLine($"\nMost Spent Category: {maxCategory.Key} ({maxCategory.Value:C})");
            }
            Console.WriteLine($"Average Monthly Savings: {GetNetSavings() / 12:C}");
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            BudgetTracker tracker = new BudgetTracker();
            bool running = true;

            while (running)
            {
                try
                {
                    Console.WriteLine("\n--- Personal Budget Tracker ---");
                    Console.WriteLine("1. Add Transaction");
                    Console.WriteLine("2. View Summary");
                    Console.WriteLine("3. View Analytics");
                    Console.WriteLine("4. Exit");
                    Console.Write("Choose an option: ");
                    int choice = int.Parse(Console.ReadLine());

                    switch (choice)
                    {
                        case 1:
                            Console.Write("Description: ");
                            string desc = Console.ReadLine();

                            Console.Write("Amount: ");
                            decimal amt = decimal.Parse(Console.ReadLine());

                            Console.Write("Type (Income/Expense): ");
                            TransactionType type = (TransactionType)Enum.Parse(typeof(TransactionType), Console.ReadLine(), true);

                            Console.Write("Category: ");
                            string cat = Console.ReadLine();

                            Console.Write("Date (yyyy-mm-dd): ");
                            DateTime date = DateTime.Parse(Console.ReadLine());

                            tracker.AddTransaction(new Transaction(desc, amt, type, cat, date));
                            break;

                        case 2:
                            Console.WriteLine($"Total Income: {tracker.GetTotalIncome():C}");
                            Console.WriteLine($"Total Expenses: {tracker.GetTotalExpenses():C}");
                            Console.WriteLine($"Net Savings: {tracker.GetNetSavings():C}");
                            break;

                        case 3:
                            tracker.DisplayTextGraph();
                            tracker.DisplayInsights();
                            break;

                        case 4:
                            running = false;
                            break;

                        default:
                            Console.WriteLine("Invalid choice.");
                            break;
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Error: {ex.Message}");
                }
            }
        }
    }

}


