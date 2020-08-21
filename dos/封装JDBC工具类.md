# 封装JDBC工具类

```java
import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.*;

/**
 * Created by Yuk on 2020/8/20.
 */
public class DBUtil {
    private static String driverClass;
    private static String url;
    private static String username_sql;
    private static String password_sql;

    private static Connection connection;
    private static PreparedStatement preparedStatement;
    private static ResultSet resultSet;

    static {
        try {
            //读取SQL配置文件 配置文件在src下
            InputStream in = DBUtil.class.getClass().getResourceAsStream("/jdbc.properties");
            Properties properties = new Properties();
            properties.load(in);
            //获取参数
            driverClass = properties.getProperty("driverClass");
            url = properties.getProperty("url");
            username_sql = properties.getProperty("username");
            password_sql = properties.getProperty("password");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 封装jdbc连接
     *
     * @return
     */
    public static Connection getConnection() {
        try {
            Class.forName(driverClass);
            connection = DriverManager.getConnection(url, username_sql, password_sql);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return connection;
    }

    /**
     * 封装jdbc增加，删除，修改
     *
     * @param sql
     * @param params
     * @return
     * @throws SQLException
     */
    public static boolean executeUpdate(String sql, Object... params) throws SQLException {
        boolean flag = false;
        try {
            int result = -1;
            connection = getConnection();
            preparedStatement = connection.prepareStatement(sql);
            int index = 1;
            if (params != null) {
                for (int i = 0; i < params.length; i++) {
                    preparedStatement.setObject(index++, params[i]);
                }
            }
            result = preparedStatement.executeUpdate();
            flag = result > 0 ? true : false;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            closeAll();
        }
        return flag;
    }

    /**
     * 封装jdbc查询方法
     *
     * @param sql
     * @param params
     * @return
     * @throws SQLException
     */
    public static List<Map<String, Object>> executeQuery(String sql, Object... params) throws SQLException {
        List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
        try {
            int index = 1;
            connection = getConnection();
            preparedStatement = connection.prepareStatement(sql);
            if (params != null) {
                for (int i = 0; i < params.length; i++) {
                    preparedStatement.setObject(index++, params[i]);
                }
            }
            resultSet = preparedStatement.executeQuery();
            ResultSetMetaData setMetaData = resultSet.getMetaData();
            // 获取列的数量
            int col_len = setMetaData.getColumnCount();
            while (resultSet.next()) {
                Map<String, Object> map = new HashMap<String, Object>();
                for (int i = 0; i < col_len; i++) {
                    String col_name = setMetaData.getColumnName(i + 1);
                    Object col_value = resultSet.getObject(col_name);
                    if (col_value == null) {
                        col_value = "";
                    }
                    map.put(col_name, col_value);
                }
                list.add(map);
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            closeAll();
        }
        return list;
    }

    /**
     * close所有的jdbc操作
     */
    public static void closeAll() {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}

```

