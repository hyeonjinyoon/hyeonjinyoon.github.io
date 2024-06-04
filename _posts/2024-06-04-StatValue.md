---
title: "[C# , Unity] StatValue"
categories:
  - Unity
tags:
  - CSharp
  - Unity
media_subpath: /assets/posts/
image:
---
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
namespace Core.Stat
{
    public enum StatModType
    {
        Add,
        MultiplierAdd,
        Multiply
    }

    public class StatModifier
    {
        public int order;
        public StatModType statModType;
        public double value;
        public object source;

        public StatModifier(StatModType statModType, double value, int order, object source)
        {
            this.statModType = statModType;
            this.value = value;
            this.order = order;
            this.source = source;
        }
    }

    public class StatValue
    {
        public double Result => _resultCache;
        public event Action OnChanged;

        private LinkedList<StatModifier> statModifiers = new();

        private double _baseValue = 0;
        private double _resultCache;

        public StatValue(double baseValue)
        {
            SetBaseValue(baseValue);
        }

        public void SetBaseValue(double baseValue)
        {
            _baseValue = baseValue;
            CalculateResult();
        }

        public void AddModifier(StatModifier statModifier)
        {
            statModifiers.AddLast(statModifier);
            CalculateResult();
        }

        public void RemoveModifier(StatModifier modifier)
        {
            statModifiers.Remove(modifier);
            CalculateResult();
        }

        public void RemoveAllModifiersFromSource(object source)
        {
            var replaceList = new LinkedList<StatModifier>();
            statModifiers.Where(modifier => modifier.source != source).Foreach(modifier => replaceList.AddLast(modifier));
            statModifiers = replaceList;
            CalculateResult();
        }

        private void CalculateResult()
        {
            _resultCache = _baseValue;

            statModifiers.GroupBy(modifier => modifier.order)
                .OrderBy(group => group.Key)
                .Foreach(group =>
                {
                    double multiplier = 1;

                    group.OrderBy(layer => layer.statModType).Foreach(modifier =>
                    {
                        switch (modifier.statModType)
                        {
                            case StatModType.Add:
                                _resultCache += modifier.value;
                                break;
                            case StatModType.MultiplierAdd:
                                multiplier += modifier.value;
                                break;
                            case StatModType.Multiply:
                                _resultCache *= modifier.value;
                                break;
                        }
                    });

                    _resultCache *= multiplier;
                });

            OnChanged?.Invoke();
        }

        public void Clear()
        {
            statModifiers.Clear();
            CalculateResult();
        }
    }
}

```


