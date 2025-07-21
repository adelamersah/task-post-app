import React, { useState, useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { View, Text, StyleSheet, TouchableOpacity, ScrollView, TextInput, Alert, Linking, FlatList, ActivityIndicator, Switch } from 'react-native';
import { MaterialIcons } from '@expo/vector-icons';
import * as SecureStore from 'expo-secure-store';

// كائن الألوان
const Colors = {
  primary: '#800020',
  secondary: '#A52A2A',
  accent: '#555555',
  lightGray: '#D3D3D3',
  background: '#F5F5F5',
  text: '#333333',
  white: '#FFFFFF',
  darkGray: '#2D2D2D',
  success: '#2E7D32',
  warning: '#F57F17',
  error: '#C62828',
};

// ==================== واجهة المستخدم (الزبائن) ====================

const HomeScreen = ({ navigation }) => {
  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.headerTitle}>Task Post - صورباهر</Text>
      </View>

      <ScrollView 
        contentContainerStyle={styles.content}
        keyboardShouldPersistTaps="handled"
      >
        <Text style={styles.welcomeText}>خدمات بريدية سريعة وموثوقة في بريد صورباهر</Text>
        
        <TouchableOpacity 
          style={[styles.serviceCard, { backgroundColor: Colors.primary }]}
          onPress={() => navigation.navigate('DeliveryRequest')}
        >
          <MaterialIcons name="local-shipping" size={32} color={Colors.white} />
          <Text style={styles.serviceTitle}>طلب توصيل طرد/معاملة</Text>
          <Text style={styles.serviceDescription}>نقوم بتوصيل طرودك ومعاملاتك بأسرع وقت</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={[styles.serviceCard, { backgroundColor: Colors.secondary }]}
          onPress={() => navigation.navigate('BusinessService')}
        >
          <MaterialIcons name="business" size={32} color={Colors.white} />
          <Text style={styles.serviceTitle}>خدمات الشركات</Text>
          <Text style={styles.serviceDescription}>حلول بريدية متكاملة لشركتك</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={[styles.serviceCard, { backgroundColor: Colors.darkGray }]}
          onPress={() => navigation.navigate('MyRequests')}
        >
          <MaterialIcons name="list-alt" size={32} color={Colors.white} />
          <Text style={styles.serviceTitle}>طلباتي</Text>
          <Text style={styles.serviceDescription}>تتبع حالة طلباتك السابقة</Text>
        </TouchableOpacity>
        
        <View style={styles.infoCard}>
          <Text style={styles.infoTitle}>معلومات التواصل</Text>
          <Text style={styles.infoText}>بريد صورباهر</Text>
          <Text style={styles.infoText}>ساعات العمل: 8:00 صباحاً - 4:00 مساءً</Text>
          
          <TouchableOpacity 
            style={styles.whatsappButton}
            onPress={() => Linking.openURL('whatsapp://send?phone=0504670703')}
          >
            <MaterialIcons name="whatsapp" size={24} color={Colors.white} />
            <Text style={styles.whatsappText}>تواصل عبر واتساب: 0504670703</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </View>
  );
};

// ==================== نماذج الطلبات ====================

const DeliveryRequestScreen = ({ navigation }) => {
  const [pickupLocation, setPickupLocation] = useState('');
  const [deliveryLocation, setDeliveryLocation] = useState('');
  const [itemDescription, setItemDescription] = useState('');
  const [notes, setNotes] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [formErrors, setFormErrors] = useState({});

  const validateForm = () => {
    const errors = {};
    if (!pickupLocation.trim()) errors.pickupLocation = 'مطلوب';
    if (!deliveryLocation.trim()) errors.deliveryLocation = 'مطلوب';
    if (!itemDescription.trim()) errors.itemDescription = 'مطلوب';
    
    setFormErrors(errors);
    return Object.keys(errors).length === 0;
  };

  const handleSubmitRequest = async () => {
    if (!validateForm()) return;
    
    setIsSubmitting(true);
    
    const newRequest = {
      id: Date.now().toString(),
      type: 'توصيل طرد',
      date: new Date().toISOString().split('T')[0],
      status: 'قيد المراجعة',
      details: {
        pickupLocation,
        deliveryLocation,
        itemDescription,
        notes,
      },
      customerType: 'individual',
    };
    
    // حفظ الطلب في التخزين المحلي
    try {
      const existingRequests = await SecureStore.getItemAsync('deliveryRequests') || '[]';
      const requests = JSON.parse(existingRequests);
      requests.push(newRequest);
      await SecureStore.setItemAsync('deliveryRequests', JSON.stringify(requests));
      
      setIsSubmitting(false);
      Alert.alert(
        'تم الإرسال بنجاح', 
        'لقد تم إرسال طلب التوصيل الخاص بك، وسنقوم بالتواصل معك قريبًا',
        [
          {
            text: 'حسناً', 
            onPress: () => {
              setPickupLocation('');
              setDeliveryLocation('');
              setItemDescription('');
              setNotes('');
              navigation.navigate('MyRequests');
            }
          }
        ]
      );
    } catch (error) {
      setIsSubmitting(false);
      Alert.alert('خطأ', 'حدث خطأ أثناء حفظ الطلب');
    }
  };

  return (
    <ScrollView 
      style={styles.container}
      keyboardShouldPersistTaps="handled"
    >
      <Text style={styles.screenTitle}>طلب توصيل طرد/معاملة</Text>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>مكان الاستلام: {formErrors.pickupLocation && <Text style={styles.errorText}>* {formErrors.pickupLocation}</Text>}</Text>
        <TextInput
          style={[styles.input, formErrors.pickupLocation && styles.inputError]}
          placeholder="مثال: منزلي، مكتب الشركة"
          placeholderTextColor={Colors.lightGray}
          value={pickupLocation}
          onChangeText={setPickupLocation}
        />
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>مكان التسليم: {formErrors.deliveryLocation && <Text style={styles.errorText}>* {formErrors.deliveryLocation}</Text>}</Text>
        <TextInput
          style={[styles.input, formErrors.deliveryLocation && styles.inputError]}
          placeholder="مثال: مركز البريد، عنوان العمل"
          placeholderTextColor={Colors.lightGray}
          value={deliveryLocation}
          onChangeText={setDeliveryLocation}
        />
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>وصف الطرد/المعاملة: {formErrors.itemDescription && <Text style={styles.errorText}>* {formErrors.itemDescription}</Text>}</Text>
        <TextInput
          style={[styles.input, styles.multilineInput, formErrors.itemDescription && styles.inputError]}
          placeholder="مثال: طرد يحتوي على كتب، أوراق تسجيل سيارة"
          placeholderTextColor={Colors.lightGray}
          value={itemDescription}
          onChangeText={setItemDescription}
          multiline
          numberOfLines={4}
        />
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>ملاحظات إضافية (اختياري):</Text>
        <TextInput
          style={[styles.input, styles.multilineInput]}
          placeholder="أوقات مفضلة للتسليم، تفاصيل إضافية"
          placeholderTextColor={Colors.lightGray}
          value={notes}
          onChangeText={setNotes}
          multiline
          numberOfLines={3}
        />
      </View>

      <TouchableOpacity 
        style={styles.submitButton} 
        onPress={handleSubmitRequest}
        disabled={isSubmitting}
      >
        {isSubmitting ? (
          <ActivityIndicator color={Colors.white} />
        ) : (
          <Text style={styles.submitButtonText}>إرسال الطلب</Text>
        )}
      </TouchableOpacity>
    </ScrollView>
  );
};

const BusinessServiceScreen = ({ navigation }) => {
  const [companyName, setCompanyName] = useState('');
  const [contactPerson, setContactPerson] = useState('');
  const [phone, setPhone] = useState('');
  const [serviceType, setServiceType] = useState('');
  const [details, setDetails] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [formErrors, setFormErrors] = useState({});

  const validateForm = () => {
    const errors = {};
    if (!companyName.trim()) errors.companyName = 'مطلوب';
    if (!contactPerson.trim()) errors.contactPerson = 'مطلوب';
    if (!phone.trim()) errors.phone = 'مطلوب';
    if (!serviceType.trim()) errors.serviceType = 'مطلوب';
    
    setFormErrors(errors);
    return Object.keys(errors).length === 0;
  };

  const handleSubmitRequest = async () => {
    if (!validateForm()) return;
    
    setIsSubmitting(true);
    
    const newRequest = {
      id: Date.now().toString(),
      type: 'خدمة تجارية',
      date: new Date().toISOString().split('T')[0],
      status: 'قيد المراجعة',
      details: {
        companyName,
        contactPerson,
        phone,
        serviceType,
        details,
      },
      customerType: 'business',
    };
    
    // حفظ الطلب في التخزين المحلي
    try {
      const existingRequests = await SecureStore.getItemAsync('businessRequests') || '[]';
      const requests = JSON.parse(existingRequests);
      requests.push(newRequest);
      await SecureStore.setItemAsync('businessRequests', JSON.stringify(requests));
      
      setIsSubmitting(false);
      Alert.alert(
        'تم الإرسال بنجاح', 
        'لقد تم إرسال طلب الخدمة التجارية الخاص بكم، وسنتواصل معكم قريبًا',
        [
          {
            text: 'حسناً', 
            onPress: () => {
              setCompanyName('');
              setContactPerson('');
              setPhone('');
              setServiceType('');
              setDetails('');
              navigation.navigate('MyRequests');
            }
          }
        ]
      );
    } catch (error) {
      setIsSubmitting(false);
      Alert.alert('خطأ', 'حدث خطأ أثناء حفظ الطلب');
    }
  };

  return (
    <ScrollView 
      style={styles.container}
      keyboardShouldPersistTaps="handled"
    >
      <Text style={styles.screenTitle}>خدمات الشركات</Text>
      
      <Text style={styles.businessDescription}>
        نقدم حلولاً بريدية متخصصة للشركات والمؤسسات في بريد صورباهر، 
        تشمل خدمات التوصيل الدورية، إدارة المراسلات، وحلول الشحن المتكاملة.
      </Text>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>اسم الشركة: {formErrors.companyName && <Text style={styles.errorText}>* {formErrors.companyName}</Text>}</Text>
        <TextInput
          style={[styles.input, formErrors.companyName && styles.inputError]}
          placeholder="اسم الشركة أو المؤسسة"
          placeholderTextColor={Colors.lightGray}
          value={companyName}
          onChangeText={setCompanyName}
        />
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>اسم المسؤول: {formErrors.contactPerson && <Text style={styles.errorText}>* {formErrors.contactPerson}</Text>}</Text>
        <TextInput
          style={[styles.input, formErrors.contactPerson && styles.inputError]}
          placeholder="اسم الشخص المسؤول"
          placeholderTextColor={Colors.lightGray}
          value={contactPerson}
          onChangeText={setContactPerson}
        />
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>رقم الهاتف: {formErrors.phone && <Text style={styles.errorText}>* {formErrors.phone}</Text>}</Text>
        <TextInput
          style={[styles.input, formErrors.phone && styles.inputError]}
          placeholder="رقم التواصل"
          placeholderTextColor={Colors.lightGray}
          value={phone}
          onChangeText={setPhone}
          keyboardType="phone-pad"
        />
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>نوع الخدمة المطلوبة: {formErrors.serviceType && <Text style={styles.errorText}>* {formErrors.serviceType}</Text>}</Text>
        <TextInput
          style={[styles.input, formErrors.serviceType && styles.inputError]}
          placeholder="مثال: توصيل دوري، إدارة مراسلات، شحن"
          placeholderTextColor={Colors.lightGray}
          value={serviceType}
          onChangeText={setServiceType}
        />
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>تفاصيل إضافية:</Text>
        <TextInput
          style={[styles.input, styles.multilineInput]}
          placeholder="وصف الخدمة المطلوبة، التكرار، أي متطلبات خاصة"
          placeholderTextColor={Colors.lightGray}
          value={details}
          onChangeText={setDetails}
          multiline
          numberOfLines={4}
        />
      </View>

      <TouchableOpacity 
        style={styles.submitButton} 
        onPress={handleSubmitRequest}
        disabled={isSubmitting}
      >
        {isSubmitting ? (
          <ActivityIndicator color={Colors.white} />
        ) : (
          <Text style={styles.submitButtonText}>إرسال الطلب</Text>
        )}
      </TouchableOpacity>
    </ScrollView>
  );
};

// ==================== واجهة المستخدم (طلباتي) ====================

const MyRequestsScreen = () => {
  const [deliveryRequests, setDeliveryRequests] = useState([]);
  const [businessRequests, setBusinessRequests] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const loadRequests = async () => {
      try {
        const deliveryData = await SecureStore.getItemAsync('deliveryRequests') || '[]';
        const businessData = await SecureStore.getItemAsync('businessRequests') || '[]';
        
        setDeliveryRequests(JSON.parse(deliveryData));
        setBusinessRequests(JSON.parse(businessData));
      } catch (error) {
        Alert.alert('خطأ', 'حدث خطأ أثناء تحميل الطلبات');
      } finally {
        setIsLoading(false);
      }
    };
    
    loadRequests();
  }, []);

  const allRequests = [...deliveryRequests, ...businessRequests].sort(
    (a, b) => new Date(b.date) - new Date(a.date)
  );

  const renderRequestItem = ({ item }) => (
    <View style={styles.requestCard}>
      <View style={styles.requestHeader}>
        <Text style={styles.requestType}>{item.type}</Text>
        <Text style={[
          styles.requestStatus,
          item.status === 'مكتمل' ? styles.completed : 
          item.status === 'قيد التنفيذ' ? styles.inProgress : 
          item.status === 'قيد المراجعة' ? styles.pending : 
          styles.review
        ]}>
          {item.status}
        </Text>
      </View>
      
      <View style={styles.requestDetails}>
        <Text style={styles.requestDetail}><Text style={styles.detailLabel}>رقم الطلب:</Text> {item.id.slice(-6)}</Text>
        <Text style={styles.requestDetail}><Text style={styles.detailLabel}>التاريخ:</Text> {item.date}</Text>
      </View>
      
      <TouchableOpacity 
        style={styles.viewDetailsButton}
        onPress={() => Alert.alert(
          'تفاصيل الطلب',
          `النوع: ${item.type}\nالحالة: ${item.status}\nالتاريخ: ${item.date}\n\n${JSON.stringify(item.details, null, 2)}`
        )}
      >
        <Text style={styles.viewDetailsText}>عرض التفاصيل</Text>
      </TouchableOpacity>
    </View>
  );

  if (isLoading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color={Colors.primary} />
        <Text style={styles.loadingText}>جاري تحميل الطلبات...</Text>
      </View>
    );
  }

  return (
    <ScrollView 
      style={styles.container}
      keyboardShouldPersistTaps="handled"
    >
      <Text style={styles.screenTitle}>طلباتي</Text>
      
      <Text style={styles.infoText}>
        قائمة بطلباتك السابقة في بريد صورباهر. يمكنك تتبع حالة كل طلب والتفاصيل المرتبطة به.
      </Text>

      {allRequests.length === 0 ? (
        <View style={styles.emptyState}>
          <MaterialIcons name="inbox" size={60} color={Colors.lightGray} />
          <Text style={styles.emptyText}>لا توجد طلبات سابقة</Text>
        </View>
      ) : (
        <FlatList
          data={allRequests}
          renderItem={renderRequestItem}
          keyExtractor={item => item.id}
          scrollEnabled={false}
          contentContainerStyle={styles.flatListContainer}
        />
      )}
    </ScrollView>
  );
};

// ==================== واجهة المطور (إدارة الطلبات) ====================

const DeveloperDashboard = () => {
  const [deliveryRequests, setDeliveryRequests] = useState([]);
  const [businessRequests, setBusinessRequests] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [activeTab, setActiveTab] = useState('delivery');

  useEffect(() => {
    const loadRequests = async () => {
      try {
        const deliveryData = await SecureStore.getItemAsync('deliveryRequests') || '[]';
        const businessData = await SecureStore.getItemAsync('businessRequests') || '[]';
        
        setDeliveryRequests(JSON.parse(deliveryData));
        setBusinessRequests(JSON.parse(businessData));
      } catch (error) {
        Alert.alert('خطأ', 'حدث خطأ أثناء تحميل الطلبات');
      } finally {
        setIsLoading(false);
      }
    };
    
    loadRequests();
  }, []);

  const updateRequestStatus = async (id, newStatus) => {
    try {
      // تحديث حالة الطلب في التوصيل
      let updatedDelivery = [...deliveryRequests];
      const deliveryIndex = updatedDelivery.findIndex(req => req.id === id);
      if (deliveryIndex !== -1) {
        updatedDelivery[deliveryIndex].status = newStatus;
        setDeliveryRequests(updatedDelivery);
        await SecureStore.setItemAsync('deliveryRequests', JSON.stringify(updatedDelivery));
      }
      
      // تحديث حالة الطلب في خدمات الشركات
      let updatedBusiness = [...businessRequests];
      const businessIndex = updatedBusiness.findIndex(req => req.id === id);
      if (businessIndex !== -1) {
        updatedBusiness[businessIndex].status = newStatus;
        setBusinessRequests(updatedBusiness);
        await SecureStore.setItemAsync('businessRequests', JSON.stringify(updatedBusiness));
      }
      
      Alert.alert('نجاح', 'تم تحديث حالة الطلب بنجاح');
    } catch (error) {
      Alert.alert('خطأ', 'حدث خطأ أثناء تحديث حالة الطلب');
    }
  };

  const renderRequestItem = ({ item }) => (
    <View style={styles.devRequestCard}>
      <View style={styles.devRequestHeader}>
        <Text style={styles.devRequestType}>{item.type}</Text>
        <Text style={styles.devRequestId}>#{item.id.slice(-6)}</Text>
      </View>
      
      <View style={styles.devRequestDetails}>
        <Text style={styles.devRequestDetail}><Text style={styles.detailLabel}>التاريخ:</Text> {item.date}</Text>
        <Text style={styles.devRequestDetail}><Text style={styles.detailLabel}>الحالة:</Text> 
          <Text style={[
            item.status === 'مكتمل' ? styles.completed : 
            item.status === 'قيد التنفيذ' ? styles.inProgress : 
            item.status === 'قيد المراجعة' ? styles.pending : 
            styles.review
          ]}>
            {item.status}
          </Text>
        </Text>
        <Text style={styles.devRequestDetail}><Text style={styles.detailLabel}>النوع:</Text> {item.customerType === 'business' ? 'شركة' : 'فرد'}</Text>
      </View>
      
      <View style={styles.statusContainer}>
        <Text style={styles.statusLabel}>تغيير الحالة:</Text>
        <View style={styles.statusButtons}>
          <TouchableOpacity 
            style={[styles.statusButton, item.status === 'قيد المراجعة' && styles.activeStatus]}
            onPress={() => updateRequestStatus(item.id, 'قيد المراجعة')}
          >
            <Text style={styles.statusButtonText}>مراجعة</Text>
          </TouchableOpacity>
          <TouchableOpacity 
            style={[styles.statusButton, item.status === 'قيد التنفيذ' && styles.activeStatus]}
            onPress={() => updateRequestStatus(item.id, 'قيد التنفيذ')}
          >
            <Text style={styles.statusButtonText}>تنفيذ</Text>
          </TouchableOpacity>
          <TouchableOpacity 
            style={[styles.statusButton, item.status === 'مكتمل' && styles.activeStatus]}
            onPress={() => updateRequestStatus(item.id, 'مكتمل')}
          >
            <Text style={styles.statusButtonText}>اكتمال</Text>
          </TouchableOpacity>
        </View>
      </View>
      
      <TouchableOpacity 
        style={styles.devViewButton}
        onPress={() => Alert.alert(
          'تفاصيل الطلب',
          JSON.stringify(item.details, null, 2)
        )}
      >
        <MaterialIcons name="visibility" size={20} color={Colors.primary} />
        <Text style={styles.devViewText}>عرض التفاصيل الكاملة</Text>
      </TouchableOpacity>
    </View>
  );

  const currentRequests = activeTab === 'delivery' ? deliveryRequests : businessRequests;

  if (isLoading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color={Colors.primary} />
        <Text style={styles.loadingText}>جاري تحميل الطلبات...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <View style={styles.devHeader}>
        <Text style={styles.devHeaderTitle}>لوحة تحكم المطور</Text>
      </View>
      
      <View style={styles.tabsContainer}>
        <TouchableOpacity 
          style={[styles.tabButton, activeTab === 'delivery' && styles.activeTab]}
          onPress={() => setActiveTab('delivery')}
        >
          <Text style={styles.tabText}>طلبات التوصيل ({deliveryRequests.length})</Text>
        </TouchableOpacity>
        <TouchableOpacity 
          style={[styles.tabButton, activeTab === 'business' && styles.activeTab]}
          onPress={() => setActiveTab('business')}
        >
          <Text style={styles.tabText}>طلبات الشركات ({businessRequests.length})</Text>
        </TouchableOpacity>
      </View>
      
      {currentRequests.length === 0 ? (
        <View style={styles.emptyState}>
          <MaterialIcons name="inbox" size={60} color={Colors.lightGray} />
          <Text style={styles.emptyText}>لا توجد طلبات</Text>
        </View>
      ) : (
        <FlatList
          data={currentRequests}
          renderItem={renderRequestItem}
          keyExtractor={item => item.id}
          contentContainerStyle={styles.devListContainer}
        />
      )}
    </View>
  );
};

// ==================== التنقل الرئيسي ====================

const Stack = createStackNavigator();

const App = () => {
  const [isDeveloperMode, setIsDeveloperMode] = useState(false);
  
  // تفعيل وضع المطور بالضغط ثلاث مرات على العنوان
  const enableDeveloperMode = () => {
    developerPressCount++;
    if (developerPressCount >= 3) {
      setIsDeveloperMode(true);
      Alert.alert('وضع المطور', 'تم تفعيل وضع المطور بنجاح');
      developerPressCount = 0;
    }
    
    setTimeout(() => { developerPressCount = 0; }, 2000);
  };
  
  let developerPressCount = 0;

  return (
    <NavigationContainer>
      <Stack.Navigator
        screenOptions={{
          headerStyle: {
            backgroundColor: Colors.primary,
          },
          headerTintColor: Colors.white,
          headerTitleStyle: {
            fontWeight: 'bold',
          },
          headerBackTitleVisible: false,
        }}
      >
        <Stack.Screen 
          name="Home" 
          component={HomeScreen} 
          options={{ 
            title: 'Task Post - صورباهر',
            headerTitle: () => (
              <TouchableOpacity onPress={enableDeveloperMode}>
                <Text style={styles.headerTitle}>Task Post - صورباهر</Text>
              </TouchableOpacity>
            )
          }} 
        />
        <Stack.Screen 
          name="DeliveryRequest" 
          component={DeliveryRequestScreen} 
          options={{ title: 'طلب توصيل' }} 
        />
        <Stack.Screen 
          name="BusinessService" 
          component={BusinessServiceScreen} 
          options={{ title: 'خدمات الشركات' }} 
        />
        <Stack.Screen 
          name="MyRequests" 
          component={MyRequestsScreen} 
          options={{ title: 'طلباتي' }} 
        />
        
        {isDeveloperMode && (
          <Stack.Screen 
            name="DeveloperDashboard" 
            component={DeveloperDashboard} 
            options={{ title: 'لوحة التحكم' }} 
          />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};

// ==================== الأنماط ====================

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: Colors.background,
  },
  header: {
    backgroundColor: Colors.primary,
    padding: 15,
    alignItems: 'center',
    justifyContent: 'center',
    elevation: 3,
    shadowColor: Colors.darkGray,
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.2,
    shadowRadius: 4,
  },
  devHeader: {
    backgroundColor: Colors.darkGray,
    padding: 15,
    alignItems: 'center',
    justifyContent: 'center',
  },
  devHeaderTitle: {
    color: Colors.white,
    fontSize: 20,
    fontWeight: 'bold',
  },
  headerTitle: {
    color: Colors.white,
    fontSize: 20,
    fontWeight: 'bold',
  },
  content: {
    padding: 20,
    paddingBottom: 30,
  },
  flatListContainer: {
    paddingBottom: 20,
  },
  devListContainer: {
    padding: 15,
  },
  welcomeText: {
    fontSize: 18,
    color: Colors.text,
    textAlign: 'center',
    marginBottom: 25,
    lineHeight: 28,
  },
  serviceCard: {
    borderRadius: 12,
    padding: 20,
    marginBottom: 15,
    alignItems: 'center',
    elevation: 2,
    shadowColor: Colors.darkGray,
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  serviceTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: Colors.white,
    marginTop: 10,
    marginBottom: 5,
  },
  serviceDescription: {
    fontSize: 15,
    color: Colors.white,
    textAlign: 'center',
    opacity: 0.9,
  },
  infoCard: {
    backgroundColor: Colors.white,
    borderRadius: 12,
    padding: 20,
    marginTop: 15,
    elevation: 2,
    shadowColor: Colors.darkGray,
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  infoTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: Colors.primary,
    marginBottom: 12,
    textAlign: 'center',
  },
  infoText: {
    fontSize: 16,
    color: Colors.text,
    marginBottom: 8,
    textAlign: 'center',
  },
  businessDescription: {
    fontSize: 16,
    color: Colors.text,
    marginBottom: 20,
    textAlign: 'center',
    lineHeight: 24,
  },
  whatsappButton: {
    flexDirection: 'row',
    backgroundColor: '#25D366',
    borderRadius: 8,
    padding: 12,
    marginTop: 15,
    alignItems: 'center',
    justifyContent: 'center',
  },
  whatsappText: {
    color: Colors.white,
    fontSize: 16,
    fontWeight: 'bold',
    marginLeft: 8,
  },
  screenTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: Colors.primary,
    marginBottom: 25,
    textAlign: 'center',
    paddingTop: 15,
  },
  inputGroup: {
    marginBottom: 20,
  },
  label: {
    fontSize: 16,
    color: Colors.text,
    marginBottom: 8,
    fontWeight: '500',
  },
  errorText: {
    color: Colors.error,
    fontSize: 14,
  },
  input: {
    borderWidth: 1,
    borderColor: Colors.lightGray,
    borderRadius: 8,
    padding: 14,
    fontSize: 16,
    backgroundColor: Colors.white,
    color: Colors.text,
  },
  inputError: {
    borderColor: Colors.error,
  },
  multilineInput: {
    minHeight: 120,
    textAlignVertical: 'top',
  },
  submitButton: {
    backgroundColor: Colors.primary,
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 10,
    marginBottom: 30,
    elevation: 2,
    shadowColor: Colors.darkGray,
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.2,
    shadowRadius: 3,
  },
  submitButtonText: {
    color: Colors.white,
    fontSize: 18,
    fontWeight: 'bold',
  },
  requestCard: {
    backgroundColor: Colors.white,
    borderRadius: 12,
    padding: 20,
    marginBottom: 15,
    elevation: 2,
    shadowColor: Colors.darkGray,
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  devRequestCard: {
    backgroundColor: Colors.white,
    borderRadius: 12,
    padding: 20,
    marginBottom: 20,
    elevation: 3,
    shadowColor: Colors.darkGray,
    shadowOffset: { width: 0, height: 3 },
    shadowOpacity: 0.2,
    shadowRadius: 5,
  },
  requestHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 10,
  },
  devRequestHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 15,
    borderBottomWidth: 1,
    borderBottomColor: Colors.lightGray,
    paddingBottom: 10,
  },
  requestType: {
    fontSize: 18,
    fontWeight: 'bold',
    color: Colors.primary,
  },
  devRequestType: {
    fontSize: 18,
    fontWeight: 'bold',
    color: Colors.primary,
  },
  devRequestId: {
    fontSize: 16,
    color: Colors.accent,
    fontWeight: 'bold',
  },
  requestStatus: {
    fontSize: 16,
    fontWeight: 'bold',
    paddingVertical: 4,
    paddingHorizontal: 10,
    borderRadius: 10,
  },
  completed: {
    backgroundColor: '#E8F5E9',
    color: '#2E7D32',
  },
  inProgress: {
    backgroundColor: '#FFF8E1',
    color: '#F57F17',
  },
  pending: {
    backgroundColor: '#FFEBEE',
    color: '#C62828',
  },
  review: {
    backgroundColor: '#E3F2FD',
    color: '#1976D2',
  },
  requestDetails: {
    marginBottom: 15,
  },
  devRequestDetails: {
    marginBottom: 15,
  },
  requestDetail: {
    fontSize: 16,
    color: Colors.text,
    marginBottom: 5,
  },
  devRequestDetail: {
    fontSize: 16,
    color: Colors.text,
    marginBottom: 8,
  },
  detailLabel: {
    fontWeight: 'bold',
    color: Colors.darkGray,
  },
  viewDetailsButton: {
    borderWidth: 1,
    borderColor: Colors.primary,
    borderRadius: 8,
    padding: 10,
    alignItems: 'center',
  },
  devViewButton: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    padding: 10,
  },
  viewDetailsText: {
    color: Colors.primary,
    fontSize: 16,
    fontWeight: 'bold',
  },
  devViewText: {
    color: Colors.primary,
    fontSize: 16,
    fontWeight: 'bold',
    marginLeft: 5,
  },
  emptyState: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 30,
  },
  emptyText: {
    fontSize: 18,
    color: Colors.lightGray,
    marginTop: 15,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  loadingText: {
    marginTop: 15,
    fontSize: 16,
    color: Colors.text,
  },
  tabsContainer: {
    flexDirection: 'row',
    margin: 15,
    backgroundColor: Colors.white,
    borderRadius: 10,
    overflow: 'hidden',
  },
  tabButton: {
    flex: 1,
    padding: 15,
    alignItems: 'center',
    backgroundColor: Colors.lightGray,
  },
  activeTab: {
    backgroundColor: Colors.primary,
  },
  tabText: {
    fontWeight: 'bold',
    color: Colors.darkGray,
  },
  statusContainer: {
    marginTop: 10,
    paddingTop: 10,
    borderTopWidth: 1,
    borderTopColor: Colors.lightGray,
  },
  statusLabel: {
    fontWeight: 'bold',
    marginBottom: 8,
    color: Colors.darkGray,
  },
  statusButtons: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  statusButton: {
    flex: 1,
    marginHorizontal: 3,
    padding: 8,
    borderRadius: 5,
    backgroundColor: Colors.lightGray,
    alignItems: 'center',
  },
  activeStatus: {
    backgroundColor: Colors.primary,
  },
  statusButtonText: {
    fontWeight: 'bold',
    color: Colors.text,
  },
});

export default App;
