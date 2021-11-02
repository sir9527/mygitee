

```java
public class HuaweiOD {

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int[] arr =new int[5];
        for (int i = 0; i <5 ; i++) {
            arr[i] = in.nextInt();
        }
        System.out.println(getNum(arr,2));
    }


    /**
     * 最大N个数和最小N个数的和
     * 测试案例通过率：76.19%
     */
    private static int getNum(int[] arr,int n){
        // 将数组去重并排序
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i <arr.length ; i++) {
            arrayList.add(arr[i]);
        }

        List<Integer> resList = arrayList.stream().distinct().sorted().collect(Collectors.toList());

        // 将前n个数放入set中
        HashSet<Integer> tmp = new HashSet<>();
        for (int i = 0; i <n ; i++) {
            tmp.add(resList.get(i));
        }


        // 将后n个数放入set中
        for (int i = resList.size()-n; i<resList.size() ; i++) {
            if (!tmp.add(resList.get(i)))return -1;
        }

        // 将最大值和最小值排序
        int res = 0;
        for (Integer a: tmp) {
            res = res + a;
        }
        return res;
    }
}

```

