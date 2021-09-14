---
title: "ScrollView实现翻页的基础操作事件"
author: 胡承
date: 2021-09-14 9:10:3 +0800
CreateTime: 2021-09-14 9:10:3 +0800
categories: C# WPF
---

逻辑分页是非常常用的一种性能优化的技术手段。虽然像`ListView`,`ListBox`有提供虚拟化的技术，能让我们加载大量数据而不会出现卡顿。但是……

<!-- more -->

实际上，虚拟化技术即便是开启后，我们在快速浏览和滚动或者是初始化加载数据的时候，会出现明显的卡顿，这并不符合我们的实际业务需要。因此，实现逻辑分页是非常有必要的。

实现逻辑分页也比较简单，主要是通过`ScrollView`来实现滚动加载，通常用于垂直滚动，分别可以像上滚动和像下滚动，水平滚动我们就不去考虑了。

常规的分页逻辑判断代码如下：

```cs
        //向上滚动
        if (e.VerticalOffset < 10 && e.VerticalChange < 0)
        {
            //do something
        }

        //向下滚动
        if ((e.ExtentHeight - (e.VerticalOffset + e.ViewportHeight)) < 10)
        {
            //do something
        }

        //滚动条是否可见，可用于针对数据模版不确定的场景，动态需要计算初始分页的条数。
        bool IsScrollBarVisible
        {
            get
            {
                return scrollViewer != null && scrollViewer.ExtentHeight > scrollViewer.ViewportHeight;
            }
        }
```

为了方便实现分页逻辑，我写了一个基于`ItemsControl`和`ScrollViewer`的基类，方便实现我们的分页逻辑：

```cs
/// <summary>
    /// 提供逻辑分页的基础实现
    /// 目标对象：ItemsControl
    /// </summary>
    public abstract class LogicPagingOperator : IDisposable
    {
        public LogicPagingOperator(ItemsControl itemsControl, int pageCapacity = 10)
        {
            ItemsSource = new ObservableCollection<object>();
            Initialize(itemsControl, pageCapacity);
        }

        protected int _pageCapacity;
        protected ScrollViewer ScrollViewer;
        protected ItemsControl ItemsControl;
        public IList OriginalSource { get; private set; }
        protected readonly ObservableCollection<object> ItemsSource;

        /// <summary>
        /// 当前页
        /// </summary>
        protected int PageIndex
        {
            get; set;
        }

        /// <summary>
        /// 逻辑源数据是否已经全部加载完成
        /// </summary>
        public bool IsLoaded
        {
            get; protected set;
        }

        private void Initialize(ItemsControl itemsControl, int pageCapacity)
        {
            ItemsControl = itemsControl;
            if (pageCapacity < 1)
            {
                ExceptionHelper.ThrowForDebug(new ArgumentException("无效的LogicPagingOperatorBase.PageCapacity值，值不能小于1"));
                pageCapacity = 1;
            }
            PageCapacity = pageCapacity;
            PageIndex = -1;
            ItemsControl.Loaded += ItemsControl_Loaded;
            ItemsControl.Unloaded += ItemsControl_Unloaded;
        }

        private void InnerReset()
        {
            ItemsSource.Clear();
            PageIndex = -1;
            IsLoaded = false;
        }

        public void SetItemsSource(IEnumerable value)
        {
            if (value == null)
            {
                OriginalSource = null;
                InnerReset();
                return;
            }

            if (value is IList list)
            {
                UnregistryNotifyCollectionChanged();

                InnerReset();

                OriginalSource = list;

                LoadNext();

                RegistryNotifyCollectionChanged();

                ItemsControl.ItemsSource = ItemsSource;
            }
            else
            {
                throw new NotSupportedException("只支持IList集合");
            }

        }

        /// <summary>
        /// 每逻辑页显示的数量
        /// </summary>
        public int PageCapacity
        {
            get
            {
                return _pageCapacity;
            }
            set
            {
                _pageCapacity = value;

                InnerReset();

                if (ItemsControl != null && ItemsControl.IsLoaded)
                {
                    LoadNext();
                }
            }
        }

        #region 数据加载

        /// <summary>
        /// 加载下一页
        /// </summary>
        public void LoadNext()
        {
            if (OriginalSource == null)
            {
                return;
            }

            if (IsLoaded)
            {
                return;
            }

            PageIndex++;
            OnLoadNext(OriginalSource);
        }

        protected virtual void OnLoadNext(IList list)
        {

        }

        #endregion

        #region 数据源变更处理

        void UnregistryNotifyCollectionChanged()
        {
            if (OriginalSource is INotifyCollectionChanged notifyCollectionChanged)
            {
                notifyCollectionChanged.CollectionChanged -= NotifyCollectionChanged_CollectionChanged;
            }
        }

        void RegistryNotifyCollectionChanged()
        {
            if (OriginalSource is INotifyCollectionChanged notifyCollectionChanged)
            {
                notifyCollectionChanged.CollectionChanged -= NotifyCollectionChanged_CollectionChanged;
                notifyCollectionChanged.CollectionChanged += NotifyCollectionChanged_CollectionChanged;
            }
        }

        void NotifyCollectionChanged_CollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
        {
            switch (e.Action)
            {
                case NotifyCollectionChangedAction.Add:
                    OnSourceAdd(e);
                    break;
                case NotifyCollectionChangedAction.Remove:
                    OnSourceRemove(e);
                    break;
                case NotifyCollectionChangedAction.Replace:
                    OnSourceReplace(e);
                    break;
                case NotifyCollectionChangedAction.Move:
                    OnSourceMove(e);
                    break;
                case NotifyCollectionChangedAction.Reset:
                    OnSourceClear(e);
                    break;
                default:
                    break;
            }
        }

        protected virtual void OnSourceAdd(NotifyCollectionChangedEventArgs e)
        {
            if (e.NewStartingIndex < ItemsSource.Count && e.NewStartingIndex > -1)
            {
                ItemsSource.Insert(e.NewStartingIndex, e.NewItems[0]);
            }
            else
            {
                foreach (var item in e.NewItems)
                {
                    ItemsSource.Add(item);
                }
            }
        }

        protected virtual void OnSourceRemove(NotifyCollectionChangedEventArgs e)
        {
            if (e.OldStartingIndex >= 0 && e.OldStartingIndex < ItemsSource.Count)
                ItemsSource.RemoveAt(e.OldStartingIndex);
        }

        protected virtual void OnSourceMove(NotifyCollectionChangedEventArgs e)
        {
            OnSourceRemove(e);
            OnSourceAdd(e);
        }

        protected virtual void OnSourceReplace(NotifyCollectionChangedEventArgs e)
        {
            if (e.NewStartingIndex < ItemsSource.Count && e.NewStartingIndex >= 0 && e.NewItems != null)
            {
                ItemsSource[e.NewStartingIndex] = e.NewItems[0];
            }
        }

        protected virtual void OnSourceClear(NotifyCollectionChangedEventArgs e)
        {
            ItemsSource.Clear();
            PageIndex = -1;
        }

        #endregion

        #region 滚动条逻辑处理

        void FindScrollView(ItemsControl itemsControl)
        {
            if (itemsControl != null && ScrollViewer == null)
            {
                ScrollViewer = itemsControl.VisualAncestor<ScrollViewer>();
                if (ScrollViewer == null)
                {
                    ScrollViewer = itemsControl.VisualDescendant<ScrollViewer>();
                }
            }
        }

        void RegistryScrollChanged(ScrollViewer scrollViewer)
        {
            if (scrollViewer != null)
            {
                scrollViewer.ScrollChanged -= ScrollViewer_ScrollChanged;
                scrollViewer.ScrollChanged += ScrollViewer_ScrollChanged;
            }
        }

        void UnregistryScrollChanged()
        {
            if (ScrollViewer != null)
            {
                ScrollViewer.ScrollChanged -= ScrollViewer_ScrollChanged;
            }
        }

        void ScrollViewer_ScrollChanged(object sender, ScrollChangedEventArgs e)
        {
            OnScrollChanged(e);
        }

        protected virtual void OnScrollChanged(ScrollChangedEventArgs e)
        {

        }

        #endregion

        #region 事件处理

        private void ItemsControl_Unloaded(object sender, System.Windows.RoutedEventArgs e)
        {
            UnregistryNotifyCollectionChanged();
            UnregistryScrollChanged();
            OnUnloaded();
        }

        private void ItemsControl_Loaded(object sender, System.Windows.RoutedEventArgs e)
        {
            FindScrollView(ItemsControl);
            RegistryNotifyCollectionChanged();
            RegistryScrollChanged(ScrollViewer);
            OnLoaded();
        }

        protected virtual void OnLoaded()
        {

        }

        protected virtual void OnUnloaded()
        {

        }

        #endregion

        public void Dispose()
        {
            ItemsSource.Clear();
            ItemsControl = null;
        }
    }

```

觉得文章好，请点个赞，给点动力^_^