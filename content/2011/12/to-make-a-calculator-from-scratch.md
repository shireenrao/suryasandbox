Title: To make a calculator from scratch
Date: 2011-12-22 14:24
Author: Surya
Category: Programming
Slug: to-make-a-calculator-from-scratch
Status: published
   
Near May 2011, as a personal challenge, I made a very basic calculator
from scratch using C\#.NET. My goals were to avoid any CLR's arithmetic
operations and integers, And to only use memory.
    
I first came up with a very basic counter, this was the building block
for Addition, Which in turn was the building block for Multiplication. I
actually have two version of addition and multiplication. The second
version has better performance as it caches intermediary results.
    
One of the cool side effect of using this technique is the results can
be bigger than Integer's Max value.  
For example it can easily calculate  
(5492012742638447789741521173787972956681294295296 \*
22867376956627844231881057173787972956681294295296)  
and return full result without the E Notation :)

    :::csharp    
    partial class FastCalculator
    {
        static LinkedList<Char> numbers = new LinkedList<Char>(new Char[] { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' });
        static Dictionary<Char, LinkedListNode<Char>> fastNumbers = new Dictionary<Char,LinkedListNode<Char>>();
        static Dictionary<string, LinkedList<char>> operationCache = new Dictionary<string, LinkedList<char>>();
      
        static FastCalculator()
        {
            LinkedListNode<Char> zero = numbers.First;
            while (zero != null)
            {
                fastNumbers[zero.Value] = zero;
                zero = zero.Next;
            }
        }
     
        static void Main(string[] args)
        {
            try
            {
                System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
                watch.Start();
        //        System.Diagnostics.Debug.WriteLine(0 + " PlusOne is " + PlusOne("0"));
        //        System.Diagnostics.Debug.WriteLine(5 + " PlusOne is " + PlusOne("5"));
        //        System.Diagnostics.Debug.WriteLine(9 + " PlusOne is " + PlusOne("9"));
        //        System.Diagnostics.Debug.WriteLine(10 + " PlusOne is " + PlusOne("10"));
        //        System.Diagnostics.Debug.WriteLine(999 + " PlusOne is " + PlusOne("999"));
        //        System.Diagnostics.Debug.WriteLine(256 + " PlusOne is " + PlusOne("256"));
        //        System.Diagnostics.Debug.WriteLine("Multiply 999*256 =" + Multiply("999", "256"));
                System.Diagnostics.Debug.WriteLine(MultiplyVer2("5492012742638447789741521173787972956681294295296", "22867376956627844231881057173787972956681294295296"));
                watch.Stop();
                TimeSpan ts = watch.Elapsed;
                string elapsedTime = String.Format("{0:00}h:{1:00}m:{2:00}s.{3:00}ms",
                                        ts.Hours, ts.Minutes, ts.Seconds,
                                        ts.Milliseconds / 10);
                System.Diagnostics.Debug.WriteLine("elapsedTime " + elapsedTime);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine(ex.Message);
            }
        }
     
        static string PlusOne(string num)
        {
            if (!string.IsNullOrEmpty(num))
            {
                LinkedList<Char> input = new LinkedList<Char>(num.ToCharArray());
                if (input.Last.Value != numbers.Last.Value)// not 9
                {
                    input.Last.Value = fastNumbers[input.Last.Value].Next.Value;
                    return LinkedListToString(input);
                }
                else // if 9
                {
                    LinkedListNode<Char> in_last = input.Last;
                    bool doCarryOver = true;
                    while (in_last != null)
                    {
                        if (in_last.Value == numbers.Last.Value)
                        {
                            if (doCarryOver)
                            {
                                in_last.Value = numbers.First.Value;
                                doCarryOver = true;
                            }
                        }
                        else
                        {
                            if (doCarryOver)
                            {
                                in_last.Value = fastNumbers[in_last.Value].Next.Value;
                                doCarryOver = false;
                            }                            
                        }
                        in_last = in_last.Previous;
                    }
                    if (doCarryOver)
                    {
                        input.AddFirst(numbers.First.Next.Value);//1
                    }
                    return LinkedListToString(input);
                }            
            }
     
            return "0";
        }
     
        static string Add(string left, string right)
        {
            string zero = numbers.First.Value.ToString();
            if (string.IsNullOrEmpty(left) && string.IsNullOrEmpty(right))
            {
                return zero;
            }
     
            while (zero != right)
            {
                left = PlusOne(left);
                zero = PlusOne(zero);
            }
            return left;
        }
     
        static string AddVer2(string left, string right)
        {
            string zero = numbers.First.Value.ToString();
            if (string.IsNullOrEmpty(left))
            {
                left = zero;
            }
     
            if (string.IsNullOrEmpty(right))
            {
                right = zero;
            }
     
            if (LengthLessThan(left, right)) //(left.Length < right.Length),make sure left is always greater
            {
                string tmp = left;
                left = right;
                right = tmp;
            }
     
            LinkedList<Char> rightIn = new LinkedList<Char>(right.ToCharArray());
            LinkedList<Char> leftIn = new LinkedList<Char>(left.ToCharArray());
            var leftInLast = leftIn.Last;
            var rightInLast = rightIn.Last;
            bool doCarryOver = false;
     
            while (leftInLast != null)
            {
                if (rightInLast != null)
                {
                    LinkedList<Char> add = AddSymbols(leftInLast.Value, rightInLast.Value);
                    if (doCarryOver)
                    {   //since carry is always 1, we will do PlusOne
                        add = new LinkedList<Char>(PlusOne(LinkedListToString(add)).ToCharArray());
                    }
                    leftInLast.Value = add.Last.Value;
                    if (add.Last.Previous != null)
                    {
                        doCarryOver = true;
                    }
                    else
                    {
                        doCarryOver = false;
                    }
                    rightInLast = rightInLast.Previous;
                }
                else
                {
                    LinkedList<Char> add = AddSymbols(leftInLast.Value, numbers.First.Value);
                    if (doCarryOver)
                    {   //since carry is always 1, we will do PlusOne
                        add = new LinkedList<Char>(PlusOne(LinkedListToString(add)).ToCharArray());
                    }
                    leftInLast.Value = add.Last.Value;
                    if (add.Last.Previous != null)
                    {
                        doCarryOver = true;
                    }
                    else
                    {
                        doCarryOver = false;
                    }
                }
     
                leftInLast = leftInLast.Previous;
            }
            if (doCarryOver)
            {
                leftIn.AddFirst(numbers.First.Next.Value);//1
            }
            return LinkedListToString(leftIn);
        }
         
        static string Multiply(string left, string right)
        {
            string zero = numbers.First.Value.ToString();
            if (string.IsNullOrEmpty(left) && string.IsNullOrEmpty(right))
            {
                return zero;
            }
     
            string rtn = zero;
     
            while (zero != right)
            {
                rtn = Add(rtn, left);
                zero = PlusOne(zero);
            }
     
            return rtn;
        }
         
        static string MultiplyVer2(string left, string right)
        {
            string zero = numbers.First.Value.ToString();
            if (string.IsNullOrEmpty(left) || string.IsNullOrEmpty(right))
            {
                return zero;
            }
     
            if (LengthLessThan(left, right))//(left.length < right.length) make sure left is always greater
            {
                var tmp = left;
                left = right;
                right = tmp;
                System.Diagnostics.Debug.WriteLine("swapped");
            }
     
            string rtn = zero;
            LinkedList<Char> rightIn = new LinkedList<Char>(right.ToCharArray());
            LinkedList<Char> leftIn = new LinkedList<Char>(left.ToCharArray());
            LinkedList<string> result2sum = new LinkedList<string>();
            var rightInLast = rightIn.Last;
            while (rightInLast != null)
            {
                var leftInLast = leftIn.Last;
                System.Diagnostics.Debug.WriteLine("right symbol " + rightInLast.Value);
                LinkedList<Char> temp = new LinkedList<char>();
                String carry = string.Empty;
                while (leftInLast != null)
                {
                    System.Diagnostics.Debug.WriteLine("left symbol " + leftInLast.Value);
                    LinkedList<Char> mult = MultiplySymbols(leftInLast.Value, rightInLast.Value);
                    if (!string.IsNullOrEmpty(carry))
                    {
                        mult = new LinkedList<Char>(AddVer2(LinkedListToString(mult), carry).ToCharArray());
                        carry = string.Empty;
                    }
                    temp.AddFirst(mult.Last.Value);
                    if (mult.Last.Previous != null)
                    {
                        carry = mult.Last.Previous.Value.ToString();
                    }
     
                    leftInLast = leftInLast.Previous;
                }
                if (!string.IsNullOrEmpty(carry))
                {
                    temp.AddFirst(Convert.ToChar(carry));
                }
                result2sum.AddFirst(LinkedListToString(temp));
                rightInLast = rightInLast.Previous;
            }
            var result = result2sum.Last;
            string sum = numbers.First.Value.ToString();//0
            string sumShift = string.Empty;
            while (result != null)
            {
                //string r = result.Value;
                System.Diagnostics.Debug.WriteLine("sum " + sum + " with " + result.Value + sumShift);
                sum = AddVer2(sum, result.Value + sumShift);
                sumShift = sumShift + numbers.First.Value.ToString();
                result = result.Previous;
            }
            return sum;
        }
     
        static LinkedList<char> AddSymbols(char left, char right)
        {
            string inputStr = left + "+" + right;
            if (operationCache.ContainsKey(inputStr))
            {
                return operationCache[inputStr];
            }
     
            string zero = numbers.First.Value.ToString();
            string rightStr = right.ToString();
            string leftStr = left.ToString();
            while (zero != rightStr)
            {
                leftStr = PlusOne(leftStr);
                zero = PlusOne(zero);
            }
            operationCache[inputStr] = new LinkedList<Char>(leftStr.ToCharArray());
            return operationCache[inputStr];
        }
         
        static LinkedList<char> MultiplySymbols(char left, char right)
        {
            string inputStr = left + "*" + right;
            if (operationCache.ContainsKey(inputStr))
            {
                return operationCache[inputStr];
            }
            string zero = numbers.First.Value.ToString();
            string rightStr = right.ToString();
            string leftStr = left.ToString();
            string rtn = zero;
            while (zero != rightStr)
            {
                rtn = AddVer2(rtn, leftStr);
                zero = PlusOne(zero);
            }
     
            operationCache[inputStr] = new LinkedList<Char>(rtn.ToCharArray());
            return operationCache[inputStr];
        }
     
        static bool LengthLessThan(string left, string right)
        {
            LinkedList<Char> rightIn = new LinkedList<Char>(right.ToCharArray());
            LinkedList<Char> leftIn = new LinkedList<Char>(left.ToCharArray());
            var leftInLast = leftIn.Last;
            var rightInLast = rightIn.Last;
            while (leftInLast != null && rightInLast != null)
            {
                leftInLast = leftInLast.Previous;
                rightInLast = rightInLast.Previous;
            }
            if (leftInLast == null && rightInLast == null)
            {
                return false;
            }
     
            if (leftInLast != null && rightInLast == null)
            {
                return false;
            }
     
            if (leftInLast == null && rightInLast != null)
            {
                return true;
            }
     
            return false;
        }
     
        private static string LinkedListToString(LinkedList<Char> num)
        {
            StringBuilder sb = new StringBuilder();
            if (num != null)
            {
                LinkedListNode<Char> first = num.First;
                while (first != null)
                {
                    sb.Append(first.Value);
                    first = first.Next;
                }
            }
     
            return sb.ToString();
        }  
    }


