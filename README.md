MemberwiseEqualityComparer
==========================

Fast (emitted IL) and foolproof way of implementing equality comparison on your objects

## The Problem ##

I grew tired of rolling my own IEquatable<T> implementations for non-essential objects. I was playing with a hobby project that required me to be able to compare the equality of objects. Performance wasn't really a factor but maintainability was and I noticed that I kept forgetting to add an equality check in my overriden Equals method when I added a new property to my object.

## Before ##

    public class Foo : IEquatable<T>
    {
        public string Bar;
        public string FooBar;
        public int FooBarId;
     
        public override int GetHashCode()
        {
            int hc = 0;
    
            if(Bar != null)
                hc ^= Bar.GetHashCode();
    
            if(FooBar != null)
                hc ^= FooBar.GetHashCode()
    
            hc ^= FooBarId.GetHashCode();
    
            return hc;
        }
     
        public override bool Equals(object obj)
        {
            if (obj == null)
                return false;
     
            return Equals(obj as Foo);
        }
     
        public override bool Equals(Foo other)
        {
            // Note the the use of ReferenceEquals to avoid infinite-recursion
            if (ReferenceEquals(other, null)
                return false;
     
            if (Bar == null && other.Bar != null)
                return false;
     
            if (!Bar.Equals(other.Bar))
                return false;
     
            if (FooBar == null && other.FooBar != null)
                return false;
     
            if (!FooBar.Equals(other.FooBar))
                return false;
     
            if (!FooBarId.Equals(other.FooBarId))
                return false;
     
            return true;
        }
    }

## After ##

    public class Foo : IEquatable<T>
    {
        public string Bar;
        public string FooBar;
        public int FooBarId;
     
        public override int GetHashCode()
        {
            return MemberwiseEqualityComparer<Foo>.Default.GetHashCode(this);
        }
     
        public override bool Equals(object obj)
        {
            if (obj == null)
                return false;
     
            return Equals(obj as Foo);
        }
     
        public override bool Equals(Foo other)
        {
            return MemberwiseEqualityComparer<Foo>.Default.Equals(this, other);
        }
    }

## Performance ##

In theory it shouldn't be possible to outperform implementing IEquatable<T> on your objects and doing every explicit check yourself but the few measurements that I've done so far suggests it's pretty darn snappy once initiated. The code is well optimized and will produce a very efficient equality comparer.

Considering the large number of pitfalls when rolling your own I see no reason not to use it, your mileage may vary ofcourse. There's a one-off performance for each type at the first call to Equals and GetHashCode when it emits IL and builds the dynamic method but after that there's really no overhead to writing the same code yourself.

As always you should try for yourself. If you have extremely performance sensitive objects that gets compared for equality like crazy then perhaps this is not for you.

## Changelog ##

### 2010-12-06 ###
 * Misc bugfixes
 * Got rid of the syncRoot object by using static initialization instead
 * Adding MemberwiseEqualityIgnoreAttribute to allow selective exclusion of fields

## License ##

The MemberwiseEqualityComparer is released under the MIT license meaining you can do pretty much whatever you want with it, including using it in proprietary solutions without changing you licensing a bit. Read more on my choice of license at my blog.