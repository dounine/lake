# 位图算法

测试

```java
@Test
    public void testBitMap(){
        int N = 1024;//位图存储的最大数字
        int map[] = new int[N/32];
        System.out.println(checkBit(map,1));
        setBit(map,1);
        setBit(map,10);
        setBit(map,4);
        System.out.println(checkBit(map,1));
        sortBitArray(map);
    }

    private boolean checkBit(int[] arr,int val){
        return (arr[val/32] & (1<<val%32))!=0;
    }
    private void setBit(int[] arr,int val){
        arr[val/32] |= 1<<val%32;
    }
    public void sortBitArray(int[] bitArray) {
        int count = 0;
        for (int i = 0; i < 1024; i++) {
            if (checkBit(bitArray, i)) {
                System.out.print(count + "\t");
            }
            count++;
        }
    }
```

结果

```java
false
true
1	4	10	
```

