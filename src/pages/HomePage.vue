<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'
import {
  BookOpenText,
  Boxes,
  BriefcaseBusiness,
  ChartNoAxesCombined,
  CheckCircle2,
  ChevronDown,
  ChevronRight,
  CircleUserRound,
  Copy,
  CornerDownLeft,
  Crosshair,
  Droplets,
  Download,
  ClipboardList,
  ClipboardPenLine,
  Clock3,
  Edit3,
  Eye,
  FlaskConical,
  Grid2X2,
  Home,
  Layers,
  Menu,
  Microscope,
  PackageCheck,
  Plus,
  Printer,
  Route,
  RotateCcw,
  Scale,
  Search,
  Send,
  Settings,
  Settings2,
  ThermometerSun,
  Trash2,
  TriangleAlert,
  X,
} from 'lucide-vue-next'

interface MenuItem {
  key: string
  label: string
  icon: typeof Home
}

interface SamplingRecord {
  orderNo: string
  sampleNo: string
  sampleName: string
  sampleCount: string
  sampleUnit?: string
  sampler: string
  samplingDate: string
  source: string
  status: string
  reason: string
  grainType: string
  warehouseNo: string
  company: string
  labelPrintStatus?: '未打印' | '已打印'
  depot: string
  sender?: string
  sendTime?: string
  receiver?: string
  receiveTime?: string
  inspectionScanTime?: string
  scanOperator?: string
  labelStatus?: string
  approvalStatus?: string
  approvalStep?: string
  reportNo?: string
  approvalHistory?: ApprovalHistory[]
  reportResults?: Record<string, ReportResult>
  reportMeta?: ReportMeta
  representativeQuantity?: string
  nature?: string
  origin?: string
  productionYear?: string
  storageDate?: string
  packageType?: string
  retainCount?: string
  inspectionCount?: string
  totalCopies?: string
  keeper?: string
  remark?: string
  cargoPosition?: string
  surveyNo?: string
  handleTime?: string
  retainTime?: string
  destroyTime?: string
  retainExpireDays?: number
  retainExpireAt?: string
  retainRecordId?: string
  retainApprovalStatus?: string
  retainApprovalStep?: '审核人' | '负责人' | '批准人' | '确认留样'
  retainApprovalHistory?: ApprovalHistory[]
  retainSampleStatus?: string
  retainHandlingMethod?: string
  retainDisposalTime?: string
  retainSpecification?: string
  destroyRecordId?: string
  destroySourceRetainId?: string
  destroyApprovalStatus?: string
  destroyApprovalStep?: '审核人' | '负责人' | '批准人' | '确认销样'
  destroyApprovalHistory?: ApprovalHistory[]
  destroyVariety?: string
  destroySource?: string
  warehouseMapSample?: boolean
  samplingApprovalStatus?: '审批中' | '已确认' | '退回修改'
  samplingApprovalStep?: '送样确认' | '收样确认'
  samplingApprovalUser?: string
  samplingApprovalHistory?: ApprovalHistory[]
}

interface SamplingForm {
  orderNo: string
  sampleNo: string
  sampleName: string
  sampleCount: string
  sampleUnit: string
  sampler: string
  samplingDate: string
  reason: string
  grainType: string
  warehouseNo: string
  company: string
  depot: string
  representativeQuantity: string
  nature: string
  origin: string
  productionYear: string
  storageDate: string
  packageType: string
  retainCount: string
  inspectionCount: string
  keeper: string
  remark: string
  cargoPosition: string
  surveyNo: string
}

interface ReportMeta {
  category: string
  compiler: string
  reviewer: string
  approver: string
  remark: string
  qualityGrade?: string
  qualified?: '是' | '否' | ''
}

interface ApprovalHistory {
  title: string
  user: string
  time: string
  opinion?: string
}

interface ReportResult {
  detectValue: string
  judgement: string
  inspector: string
  standard?: string
  type?: string
  isKey?: boolean
  instrumentCode?: string
  instrumentName?: string
  capturedAt?: string
}

interface QualityStandard {
  item: string
  standard: string
  isKey: boolean
  type: string
}

interface UnqualifiedItem {
  item: string
  detectValue: string
  judgement: string
  standard: string
  type: string
  isKey: boolean
}

type QualityRiskCode = 'normal' | 'level1' | 'level2' | 'level3'
type QualityWarningLevelFilter = 'all' | Exclude<QualityRiskCode, 'normal'>
type InspectionOverviewDrilldown = 'sampling' | 'pending' | 'approval' | 'retain' | 'destroy'
type InspectionTodoCategory = 'send' | 'receive' | 'inspect' | 'result' | 'approval' | 'destroy'
type ReportViewMode = 'result' | 'report'

interface SystemUser {
  username: string
  password: string
  name: string
  role: string
  department: string
  phone: string
  status: string
}

interface UserForm extends SystemUser {}

interface LabInstrument {
  code: string
  name: string
  model: string
  location: string
  custodian: string
  status: string
  calibration: string
  maintenance: string
  lastMaintenance: string
  maintenanceType: string
  maintenanceHandler: string
  abandonedAt?: string
  abandonHandler?: string
  usage: number
  type: string
  lastCalibration: string
}

interface LabInstrumentCategory {
  type: string
  description: string
  storageArea: string
  note: string
}

interface LabCalibrationHistoryRecord {
  recordNo: string
  instrumentCode: string
  instrumentName: string
  actualDate: string
  planDate: string
  agency: string
  certificate: string
  result: string
}

interface ReagentStock {
  code: string
  name: string
  type: string
  spec: string
  stock: number
  unit: string
  threshold: number
  location: string
  keeper: string
  status: string
}

interface ReagentFlowHistory {
  title: string
  user: string
  time: string
}

type ReagentApprovalStep = '编制人' | '审核人' | '批准人'
type ReagentActionType = '采购' | '入库' | '领用' | '归还' | '盘点'

interface ReagentFlowRecord {
  recordNo: string
  action: ReagentActionType
  reagentCode: string
  reagentName: string
  quantity: number
  unit: string
  location: string
  handler: string
  time: string
  remark: string
  supplier?: string
  approvalStatus: string
  approvalStep?: ReagentApprovalStep
  approvalHistory?: ReagentFlowHistory[]
  processStatus?: '审批中' | '领用中' | '已归还' | '待盘点' | '已盘点'
  sourceRecordNo?: string
  returnRecordNo?: string
  inventoryRecordNo?: string
  stockApplied?: boolean
  inventoryStatus?: '待盘点' | '已盘点'
  inventoryActualQuantity?: number
  inventoryResult?: '账实相符' | '存在差异'
  inventoryConfirmedBy?: string
  inventoryConfirmedAt?: string
  inventoryRemark?: string
}

interface ReagentProcessForm {
  action: ReagentActionType
  reagentCode: string
  name: string
  spec: string
  unit: string
  quantity: number
  threshold: number
  location: string
  keeper: string
  handler: string
  time: string
  type: string
  remark: string
  supplier: string
}

interface ReagentCabinetEnvironment {
  code: string
  name: string
  type: '普通试剂柜' | '易制毒双人双锁柜'
  temperature: number
  humidity: number
  sampledAt: string
}

interface ReagentCabinetRecord {
  time: string
  cabinetCode: string
  cabinetName: string
  temperature: number
  humidity: number
  status: string
}

interface EnvironmentRecord {
  time: string
  area: string
  temperature: number
  humidity: number
  status: string
}

type EnvironmentRange = 'day' | 'week' | 'month' | 'halfYear'
type EnvironmentAlarmType = 'all' | 'normal' | 'temperature' | 'humidity' | 'both'

interface EnvironmentChartPoint {
  label: string
  date: string
  temperature: number
  humidity: number
  x: number
  temperatureY: number
  humidityY: number
}

interface WarehouseMarkerCoordinate {
  x: number
  y: number
}

interface PersistedState {
  internalSamplingRecords: SamplingRecord[]
  externalSamplingRecords: SamplingRecord[]
  retainRecords?: SamplingRecord[]
  destroyRecords?: SamplingRecord[]
  systemUsers: SystemUser[]
  deletedLabelNos: string[]
  reagentStocks?: ReagentStock[]
  reagentFlowRecords?: ReagentFlowRecord[]
  reagentCabinets?: ReagentCabinetEnvironment[]
}

const menuItems: MenuItem[] = [
  { key: 'dashboard', label: '质量看板', icon: Grid2X2 },
  { key: 'inspection', label: '检验流程管理', icon: ClipboardList },
  { key: 'warning', label: '质量预警记录', icon: TriangleAlert },
  { key: 'report', label: '质量报表管理', icon: BookOpenText },
  { key: 'lab', label: '检化验室管理', icon: BriefcaseBusiness },
  { key: 'reagent', label: '检化验试剂管理', icon: Boxes },
  { key: 'system', label: '用户管理', icon: Settings },
]

const activeMenu = ref('dashboard')
const selectedWarehouse = ref('51仓')
const activeWarehousePositionIndex = ref(0)
const warehouseMapShell = ref<HTMLElement | null>(null)
const warehouseMapCalibrationMode = ref(false)
const warehouseMapCalibrationHover = ref<WarehouseMarkerCoordinate | null>(null)
const warehouseMapCalibrationSelectedIndex = ref<number | null>(null)
const warehouseMapCalibrationDragIndex = ref<number | null>(null)
const warehouseMapCalibrationNotice = ref('')
const showProcessDialog = ref(false)
const inspectionTab = ref('overview')
const inspectionOverviewDrilldown = ref<InspectionOverviewDrilldown | null>(null)
const inspectionTodoDialog = ref<InspectionTodoCategory | null>(null)
const inspectionOverviewTodayDate = ref('2026-07-04')
const inspectionOverviewFilters = ref({ keyword: '', warehouseNo: '', source: '', reason: '', retainStatus: '' })
const samplingSource = ref<'internal' | 'external'>('internal')
const samplingReason = ref('入库验收')
const showSamplingForm = ref(false)
const showSamplingDetail = ref(false)
const showSamplingPreview = ref(false)
const showSamplingLabel = ref(false)
const showSendSample = ref(false)
const showReceiveSample = ref(false)
const showSamplingApprovalFlow = ref(false)
const samplingApprovalOpinion = ref('')
const showInspectionScanDialog = ref(false)
const inspectionScanValue = ref('')
const showReportForm = ref(false)
const showInstrumentDataDialog = ref(false)
const selectedInstrumentCodes = ref<string[]>([])
const showConfirmDialog = ref(false)
const confirmDialog = ref({ title: '操作确认', message: '', confirmText: '确认', danger: false })
const pendingConfirmAction = ref<(() => void) | null>(null)
const showReportView = ref(false)
const reportViewMode = ref<ReportViewMode>('report')
const showApprovalFlow = ref(false)
const approvalOpinion = ref('')
const showRetainScanDialog = ref(false)
const retainScanValue = ref('')
const showRetainForm = ref(false)
const showRetainApprovalFlow = ref(false)
const selectedRetainRecordNo = ref('')
const retainApprovalOpinion = ref('')
const retainForm = ref({ recordId: '', sampleNo: '', sampleName: '', sampleDate: '', sampleCount: '', sampleUnit: 'kg', specification: '', warehouseNo: '', cargoPosition: '', expireAt: '', sampleStatus: '审批中', handlingMethod: '留存', disposalTime: '', sourceRecordNo: '', remark: '' })
const internalRetainRecords = ref<SamplingRecord[]>([])
const internalDestroyRecords = ref<SamplingRecord[]>([])
const showDestroyForm = ref(false)
const showDestroyApprovalFlow = ref(false)
const selectedDestroyRecordNo = ref('')
const destroyApprovalOpinion = ref('')
const destroyForm = ref({ recordId: '', sourceRetainId: '', sampleNo: '', sampleName: '', warehouseNo: '', cargoPosition: '', specification: '', variety: '', destroySource: '留样到期', remark: '' })
const upcomingBusinessDialog = ref<'retain' | 'destroy' | null>(null)
const upcomingBusinessRecord = ref<SamplingRecord | null>(null)

const showInstrumentCategoryForm = ref(false)
const showInstrumentEntryForm = ref(false)
const instrumentCategoryForm = ref<LabInstrumentCategory>({
  type: '',
  description: '',
  storageArea: '',
  note: '',
})
const instrumentEntryForm = ref({
  type: '',
  name: '',
  model: '',
  location: '',
  custodian: '',
  calibration: '',
})
const selectedCalibrationInstrumentCode = ref('')
const showCalibrationHistoryDialog = ref(false)
const showMaintenanceForm = ref(false)
const selectedMaintenanceInstrumentCode = ref('')
const maintenanceForm = ref({
  date: '2026-07-12',
  type: '月度维护',
  handler: '',
  nextDate: '',
})
const showAbandonForm = ref(false)
const selectedAbandonInstrumentCode = ref('')
const abandonForm = ref({ date: '2026-07-12', handler: '' })
const showEnvironmentTrendDialog = ref(false)

const showReagentEntryForm = ref(false)
const showReagentInboundReport = ref(false)
const selectedReagentInboundRecord = ref<ReagentFlowRecord | null>(null)
const showReagentInventoryConfirmDialog = ref(false)
const selectedReagentInventoryRecord = ref<ReagentFlowRecord | null>(null)
const reagentInventoryConfirmReadOnly = ref(false)
const reagentInventoryConfirmForm = ref({ actualQuantity: 0, confirmer: '', remark: '' })
const showReagentCabinetForm = ref(false)
const showReagentApprovalFlow = ref(false)
const selectedReagentRecordNo = ref('')
const reagentProcessTab = ref<ReagentActionType>('入库')
const reagentProcessTabs: ReagentActionType[] = ['采购', '入库', '领用', '归还', '盘点']
const showReagentFlowForm = ref(false)
const reagentFlowForm = ref<ReagentProcessForm>({
  action: '入库',
  reagentCode: '',
  name: '',
  spec: '',
  unit: '瓶',
  quantity: 0,
  threshold: 5,
  location: '',
  keeper: '',
  handler: '',
  time: '2026-07-12T09:00',
  type: '普通试剂',
  remark: '',
  supplier: '',
})
const reagentFlowSourceRecordNo = ref('')
const selectedReagentCabinetCode = ref('A01')
const activeReagentCabinetType = ref<ReagentCabinetEnvironment['type']>('普通试剂柜')
const showReagentCabinetHistory = ref(false)
const showReagentCabinetTrendDialog = ref(false)
const activeReagentCabinetRange = ref<EnvironmentRange>('day')
const activeReagentCabinetFilter = ref('全部试剂柜')
const reagentCabinetStockKeyword = ref('')
const reagentCabinetStockStatusFilter = ref<'all' | 'normal' | 'abnormal'>('all')
const hoveredReagentCabinetPoint = ref<EnvironmentChartPoint | null>(null)
const reagentEntryForm = ref<ReagentProcessForm>({
  action: '入库',
  reagentCode: '',
  name: '',
  spec: '',
  unit: '瓶',
  quantity: 0,
  threshold: 5,
  location: '',
  keeper: '',
  handler: '',
  time: '2026-07-12T09:00',
  type: '普通试剂',
  remark: '',
  supplier: '',
})
const reagentCabinetForm = ref({
  code: '',
  name: '',
  type: '普通试剂柜' as ReagentCabinetEnvironment['type'],
  temperature: 22,
  humidity: 45,
})

const reportFilters = ref({ sampleNo: '', sampleName: '', processStatus: '' })
const warningLevelFilter = ref<QualityWarningLevelFilter>('all')
const warningFilters = ref({ keyword: '', warehouseNo: '', reason: '', date: '' })
const showUnqualifiedDialog = ref(false)
const showUserForm = ref(false)
const selectedSamplingNo = ref('QY20260701001')
const deletedLabelNos = ref<string[]>([])
const loginError = ref('')
const labelFilter = ref({ sampleName: '', source: '', sampleNo: '', printStatus: '' })
const samplingFilter = ref({ orderNo: '', sampleName: '', reason: '', source: '', status: '' })
const ledgerFilter = ref({ grainType: '', source: '', reason: '', startDate: '', endDate: '' })
const activeQualityReport = ref('cargo')
const activeWarehouseReportGrain = ref<'玉米' | '大豆'>('玉米')
const labTab = ref('ledger')
const reagentTab = ref('overview')
const activeInstrumentType = ref('物理检测设备')
const activeEnvironmentRange = ref<EnvironmentRange>('day')
const activeEnvironmentArea = ref('全部区域')
const activeEnvironmentAlarmType = ref<EnvironmentAlarmType>('all')
const hoveredEnvironmentPoint = ref<EnvironmentChartPoint | null>(null)
const environmentChartWidth = 860
const environmentChartHeight = 320
const environmentChartPaddingX = 64
const environmentChartPaddingY = 28
const environmentChartLabelY = 306
const loginForm = ref({ username: 'wangshuai', password: '123456' })
const currentLoginUser = ref<SystemUser | null>(null)
const samplingFormMode = ref<'create' | 'edit'>('create')
const samplingForm = ref<SamplingForm>({ orderNo: '', sampleNo: '', sampleName: '', sampleCount: '2kg × 4', sampleUnit: 'kg', sampler: '张三', samplingDate: '2026-07-04', reason: '入库验收', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '江苏', productionYear: '2026', storageDate: '2026-06-17', packageType: '散装', retainCount: '1', inspectionCount: '1', keeper: '王保管', remark: '样品用于质量检验流程演示。', cargoPosition: '49仓1号货位', surveyNo: '' })
const sendSampleForm = ref({ sender: '王帅', sendTime: '2026-07-19T10:30' })
const receiveSampleForm = ref({ receiver: '质检员A', receiveTime: '2026-07-04T11:10' })
const reportMetaForm = ref<ReportMeta>({ category: '监督检验', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '', qualityGrade: '一等', qualified: '是' })
const userFormMode = ref<'create' | 'edit'>('create')
const userForm = ref<UserForm>({ username: '', password: '123456', name: '', role: '质检员', department: '质量管理科', phone: '', status: '启用' })
const storageKey = 'quality-management-grain-standard-state-v2'

const systemUsers = ref<SystemUser[]>([
  { username: 'fuzeren', password: '123456', name: '\u5468\u8d1f\u8d23', role: '\u8d1f\u8d23\u4eba', department: '\u8d28\u91cf\u7ba1\u7406\u79d1', phone: '13800000005', status: '\u542f\u7528' },
  { username: 'wangshuai', password: '123456', name: '王帅', role: '编制人', department: '质量管理科', phone: '13800000001', status: '启用' },
  { username: 'lishenhe', password: '123456', name: '李审核', role: '审核人', department: '质量管理科', phone: '13800000002', status: '启用' },
  { username: 'zhaopizhun', password: '123456', name: '赵批准', role: '批准人', department: '库领导', phone: '13800000003', status: '启用' },
  { username: 'zhijianyuan', password: '123456', name: '质检员A', role: '质检员', department: '检化验室', phone: '13800000004', status: '启用' },
])

const handleLogin = () => {
  const matchedUser = systemUsers.value.find((item) => item.username === loginForm.value.username && item.password === loginForm.value.password && item.status === '启用')
  if (matchedUser) {
    currentLoginUser.value = matchedUser
    loginError.value = ''
  } else {
    currentLoginUser.value = null
    loginError.value = '用户名、密码错误或账号已停用'
  }
}

const handleLogout = () => {
  currentLoginUser.value = null
  activeMenu.value = 'dashboard'
}

const breadcrumbCurrent = computed(() => {
  return activeMenu.value === 'inspection' ? '检验流程管理' : '质量看板'
})

const activeSamplingRecords = computed(() => {
  const records = samplingSource.value === 'internal' ? internalSamplingRecords.value : externalSamplingRecords.value
  return records.filter((item) => {
    const matchedOrderNo = !samplingFilter.value.orderNo || item.orderNo.includes(samplingFilter.value.orderNo)
    const matchedName = !samplingFilter.value.sampleName || item.sampleName.includes(samplingFilter.value.sampleName)
    const matchedReason = !samplingFilter.value.reason || item.reason === samplingFilter.value.reason
    const matchedSource = !samplingFilter.value.source || item.source === samplingFilter.value.source
    const matchedStatus = !samplingFilter.value.status || item.status === samplingFilter.value.status
    return matchedOrderNo && matchedName && matchedReason && matchedSource && matchedStatus
  })
})

const allSamplingRecords = computed(() => [...internalSamplingRecords.value, ...externalSamplingRecords.value])

const samplingFlowSummary = computed(() => {
  const records = activeSamplingRecords.value
  return [
    { label: '录入中', value: records.filter((record) => record.status === '录入中').length },
    { label: '扦样完成', value: records.filter((record) => record.status === '扦样完成').length },
    { label: '已送样', value: records.filter((record) => record.status === '已送样').length },
    { label: '已收样', value: records.filter((record) => record.status === '已收样').length },
  ]
})

const selectedSamplingRecord = computed(() => {
  return allSamplingRecords.value.find((item) => item.orderNo === selectedSamplingNo.value) ?? activeSamplingRecords.value[0] ?? allSamplingRecords.value[0]
})

const previewSamplingRecord = computed<SamplingRecord>(() => {
  if (showSamplingForm.value) {
    return { ...samplingForm.value, source: samplingSource.value === 'internal' ? '内部扦样' : '外部扦样', status: samplingFormMode.value === 'create' ? '录入中' : selectedSamplingRecord.value?.status ?? '录入中', reason: samplingReason.value }
  }
  return selectedSamplingRecord.value
})

const labelRecords = computed(() => {
  return allSamplingRecords.value.filter((item) => {
    const matchedStatus = item.status === '已收样'
    const matchedDeleted = !deletedLabelNos.value.includes(item.sampleNo)
    const matchedName = !labelFilter.value.sampleName || item.sampleName.includes(labelFilter.value.sampleName)
    const matchedSource = !labelFilter.value.source || item.source === labelFilter.value.source
    const matchedNo = !labelFilter.value.sampleNo || item.sampleNo.includes(labelFilter.value.sampleNo)
    const matchedPrintStatus = !labelFilter.value.printStatus || (item.labelPrintStatus ?? '未打印') === labelFilter.value.printStatus
    return matchedStatus && matchedDeleted && matchedName && matchedSource && matchedNo && matchedPrintStatus
  })
})

const labelPrintStatusText = (record: SamplingRecord) => record.labelPrintStatus ?? '未打印'

const selectedLabelRecord = computed(() => {
  return labelRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value) ?? labelRecords.value[0]
})

const reportRecords = computed(() => {
  return allSamplingRecords.value.filter((item) => item.status === '已收样')
})

const filteredReportRecords = computed(() => {
  const filters = reportFilters.value
  return reportRecords.value.filter((item) => {
    if (filters.sampleNo && !item.sampleNo.includes(filters.sampleNo)) return false
    if (filters.sampleName && !item.sampleName.includes(filters.sampleName)) return false
    if (filters.processStatus && reportProcessStatusText(item) !== filters.processStatus) return false
    return true
  })
})

const selectedReportRecord = computed(() => {
  return reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value) ?? reportRecords.value[0]
})

const selectedUnqualifiedRecord = computed(() => {
  return reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value) ?? reportRecords.value[0]
})

const retainRecords = computed(() => [
  ...internalRetainRecords.value,
  ...reportRecords.value.filter((item) => item.labelStatus === '留样' && !internalRetainRecords.value.some((retain) => retain.sampleNo === item.sampleNo)),
])
const destroyRecords = computed(() => [
  ...internalDestroyRecords.value,
  ...reportRecords.value.filter((item) => item.labelStatus === '销样' && !internalDestroyRecords.value.some((destroy) => destroy.sampleNo === item.sampleNo)),
])

const reportUnqualifiedMap: Record<string, string[]> = {
  YP20260701010: ['水分及挥发物(%)', '酸价(KOH)/(mg/g)'],
  YP20260701012: ['水分及挥发物(%)', '黄曲霉毒素B1(μg/kg)', '酸价(KOH)/(mg/g)', '铅(mg/kg)'],
  WYP20260701007: ['铅(mg/kg)'],
  YP20260705002: ['水分 (g/100g)', '霉变粒率 (%)'],
  YP20260706002: ['霉变粒率 (%)'],
  YP20260706004: ['热损伤粒率 (%)', '破碎粒率 (%)'],
}

const reportStandardMeta = (record: SamplingRecord, item: string) => activeQualityStandards(record).find((standard) => standard.item === item)

const buildUnqualifiedItem = (record: SamplingRecord, item: string, result?: ReportResult): UnqualifiedItem => {
  const standard = reportStandardMeta(record, item)
  return {
    item,
    detectValue: result?.detectValue ?? (standard ? `${standard.standard}，超标准值` : ''),
    judgement: result?.judgement ?? '不合格',
    standard: result?.standard ?? standard?.standard ?? '',
    type: result?.type ?? standard?.type ?? '检验指标',
    isKey: result?.isKey ?? standard?.isKey ?? false,
  }
}

const reportUnqualifiedDetails = (record: SamplingRecord) => {
  if (record.reportResults) {
    return Object.entries(record.reportResults)
      .filter(([, result]) => result.judgement === '不合格')
      .map(([item, result]) => buildUnqualifiedItem(record, item, result))
  }
  return (reportUnqualifiedMap[record.sampleNo] ?? []).map((item) => buildUnqualifiedItem(record, item))
}

const reportUnqualifiedItems = (record: SamplingRecord) => reportUnqualifiedDetails(record).map((item) => item.item)
const reportUnqualifiedCount = (record: SamplingRecord) => reportUnqualifiedItems(record).length
const reportHasUnqualified = (record: SamplingRecord) => reportUnqualifiedCount(record) > 0

const currentWarehouseSampleRecords = computed(() => {
  const groupedRecords = new Map<string, SamplingRecord[]>()
  reportRecords.value.forEach((record) => {
    const cargoPosition = normalizedCargoPosition(record)
    groupedRecords.set(cargoPosition, [...(groupedRecords.get(cargoPosition) ?? []), record])
  })

  const currentRecordsByPosition = [...groupedRecords.values()].map((records) => {
    return [...records].sort((a, b) => {
      const mapPriority = Number(b.warehouseMapSample) - Number(a.warehouseMapSample)
      if (mapPriority) return mapPriority
      const aTime = a.receiveTime ?? a.samplingDate
      const bTime = b.receiveTime ?? b.samplingDate
      return bTime.localeCompare(aTime)
    })[0]
  })

  const groupedByWarehouse = new Map<string, SamplingRecord[]>()
  currentRecordsByPosition.forEach((record) => {
    groupedByWarehouse.set(record.warehouseNo, [...(groupedByWarehouse.get(record.warehouseNo) ?? []), record])
  })

  return [...groupedByWarehouse.values()].map((records) => {
    return [...records].sort((a, b) => {
      const mapPriority = Number(b.warehouseMapSample) - Number(a.warehouseMapSample)
      if (mapPriority) return mapPriority
      const aTime = a.receiveTime ?? a.samplingDate
      const bTime = b.receiveTime ?? b.samplingDate
      return bTime.localeCompare(aTime)
    })[0]
  })
})

const currentWarehouseSampleRecordMap = computed(() => new Map(currentWarehouseSampleRecords.value.map((record) => [record.warehouseNo, record])))

const qualityRiskLevel = (record: SamplingRecord) => {
  const items = reportUnqualifiedDetails(record)
  const keyCount = items.filter((item) => item.isKey).length
  const normalCount = items.length - keyCount
  const total = items.length
  let code: QualityRiskCode = 'normal'
  let label = '正常'
  let shortLabel = '正常'
  let status = '未发现不合格项目，质量检验正常'

  if (keyCount >= 2 || (keyCount === 1 && normalCount >= 2)) {
    code = 'level1'
    label = '一级预警'
    shortLabel = '一级'
    status = `共${total}项不合格，触发一级预警`
  } else if ((keyCount === 0 && normalCount >= 3) || (keyCount === 1 && normalCount <= 1)) {
    code = 'level2'
    label = '二级预警'
    shortLabel = '二级'
    status = `共${total}项不合格，触发二级预警`
  } else if (keyCount === 0 && normalCount >= 1 && normalCount <= 2) {
    code = 'level3'
    label = '三级预警'
    shortLabel = '三级'
    status = `共${total}项不合格，触发三级预警`
  }

  return { code, label, shortLabel, status, keyCount, normalCount, total }
}

const openUnqualifiedItems = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  showUnqualifiedDialog.value = true
}

const qualityWarningTime = (record: SamplingRecord) => {
  const capturedTimes = Object.values(record.reportResults ?? {})
    .filter((result) => result.judgement === '不合格' && result.capturedAt)
    .map((result) => String(result.capturedAt))
    .sort((a, b) => b.localeCompare(a))
  return capturedTimes[0]
    ?? record.approvalHistory?.[record.approvalHistory.length - 1]?.time
    ?? record.handleTime
    ?? record.inspectionScanTime
    ?? record.receiveTime
    ?? record.samplingDate
}

const qualityWarningRecords = computed(() => {
  return currentWarehouseSampleRecords.value
    .map((record) => {
      const items = reportUnqualifiedDetails(record)
      const risk = qualityRiskLevel(record)
      return {
        record,
        items,
        risk,
        count: risk.total,
        level: risk.label,
        status: risk.status,
        warningTime: qualityWarningTime(record),
      }
    })
    .filter((item) => item.risk.code !== 'normal')
})

const warningLevelTags = computed<{ key: QualityWarningLevelFilter; label: string; count: number }[]>(() => [
  { key: 'all', label: '全部预警', count: qualityWarningRecords.value.length },
  { key: 'level1', label: '一级预警', count: qualityWarningRecords.value.filter((item) => item.risk.code === 'level1').length },
  { key: 'level2', label: '二级预警', count: qualityWarningRecords.value.filter((item) => item.risk.code === 'level2').length },
  { key: 'level3', label: '三级预警', count: qualityWarningRecords.value.filter((item) => item.risk.code === 'level3').length },
])

const warningWarehouseOptions = computed(() => [...new Set(qualityWarningRecords.value.map((item) => item.record.warehouseNo))].sort((a, b) => Number(a) - Number(b)))
const warningReasonOptions = computed(() => [...new Set(qualityWarningRecords.value.map((item) => item.record.reason))].sort())

const warningDateText = (value: string) => {
  const [year = '', month = '', day = ''] = value.replace(/[./]/g, '-').split(' ')[0].split('-')
  return year && month && day ? `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}` : value
}

const filteredQualityWarningRecords = computed(() => {
  const keyword = warningFilters.value.keyword.trim().toLowerCase()
  return qualityWarningRecords.value.filter((item) => {
    const record = item.record
    if (warningLevelFilter.value !== 'all' && item.risk.code !== warningLevelFilter.value) return false
    if (keyword && ![record.sampleNo, record.sampleName, record.reportNo ?? ''].some((value) => value.toLowerCase().includes(keyword))) return false
    if (warningFilters.value.warehouseNo && record.warehouseNo !== warningFilters.value.warehouseNo) return false
    if (warningFilters.value.reason && record.reason !== warningFilters.value.reason) return false
    if (warningFilters.value.date && warningDateText(item.warningTime) !== warningFilters.value.date) return false
    return true
  })
})

const resetWarningFilters = () => {
  warningLevelFilter.value = 'all'
  warningFilters.value = { keyword: '', warehouseNo: '', reason: '', date: '' }
}

const qualityWarningStats = computed(() => {
  const rows = currentWarehouseSampleRecords.value.map((record) => qualityRiskLevel(record).code)
  return {
    normal: rows.filter((code) => code === 'normal').length,
    level1: rows.filter((code) => code === 'level1').length,
    level2: rows.filter((code) => code === 'level2').length,
    level3: rows.filter((code) => code === 'level3').length,
  }
})

const currentWarningCount = computed(() => qualityWarningStats.value.level1 + qualityWarningStats.value.level2 + qualityWarningStats.value.level3)

const totalQualityRecords = computed(() => qualityWarningStats.value.normal + qualityWarningStats.value.level1 + qualityWarningStats.value.level2 + qualityWarningStats.value.level3)

const level1Rate = computed(() => totalQualityRecords.value ? Math.round((qualityWarningStats.value.level1 / totalQualityRecords.value) * 100) : 0)
const level2Rate = computed(() => totalQualityRecords.value ? Math.round((qualityWarningStats.value.level2 / totalQualityRecords.value) * 100) : 0)
const level3Rate = computed(() => totalQualityRecords.value ? Math.round((qualityWarningStats.value.level3 / totalQualityRecords.value) * 100) : 0)
const normalRate = computed(() => Math.max(0, 100 - level1Rate.value - level2Rate.value - level3Rate.value))

const warningDonutStyle = computed(() => ({
  background: `conic-gradient(#2f80ed 0 ${normalRate.value}%, #eb5757 ${normalRate.value}% ${normalRate.value + level1Rate.value}%, #f2994a ${normalRate.value + level1Rate.value}% ${normalRate.value + level1Rate.value + level2Rate.value}%, #f2c94c ${normalRate.value + level1Rate.value + level2Rate.value}% 100%)`,
}))

const warningWarehouseCount = computed(() => new Set(qualityWarningRecords.value.map((item) => normalizedCargoPosition(item.record))).size)
const warningUnqualifiedTotal = computed(() => qualityWarningRecords.value.reduce((total, item) => total + item.count, 0))

const warningTypes = computed(() => {
  const rows = [
    { label: '一级预警', count: qualityWarningStats.value.level1 },
    { label: '二级预警', count: qualityWarningStats.value.level2 },
    { label: '三级预警', count: qualityWarningStats.value.level3 },
  ]
  const maxCount = Math.max(...rows.map((item) => item.count), 1)
  return rows.map(({ label, count }) => ({
    label,
    count,
    percent: Math.max(18, Math.round((count / maxCount) * 100)),
  }))
})

const warningRecords = computed(() => qualityWarningRecords.value.map((item) => ({
  level: item.risk.shortLabel,
  levelCode: item.risk.code,
  warehouse: `${item.record.warehouseNo}仓`,
  position: normalizedCargoPosition(item.record),
  sampleName: item.record.sampleName,
  content: `${item.count}项不合格`,
  status: item.status,
  sampleNo: item.record.sampleNo,
})))

const openQualityWarningRecord = (sampleNo: string, warehouse: string) => {
  selectedSamplingNo.value = sampleNo
  selectedWarehouse.value = warehouse
  activeMenu.value = 'warning'
}

const qualityReportTabs = [
  { key: 'cargo', label: '质量分析货位明细表' },
  { key: 'warehouse', label: '分货位质量汇总表' },
  { key: 'acceptance', label: '质量分析验收申请函' },
  { key: 'season', label: '春秋普检测结果汇总表' },
]

const completedReportRecords = computed(() => reportRecords.value.filter((record) => record.approvalStatus === '审批通过' && Boolean(record.reportNo)))

const displaySampleNo = (record: SamplingRecord) => record.sampleNo?.trim() || record.orderNo || '—'

const cargoPositionOptions = Array.from({ length: 30 }, (_, index) => {
  const warehouseNo = String(index + 49)
  return { label: `${warehouseNo}仓1号货位`, warehouseNo }
})

const cargoPositionByWarehouse: Record<string, string[]> = cargoPositionOptions.reduce<Record<string, string[]>>((result, item) => {
  result[item.warehouseNo] = [...(result[item.warehouseNo] ?? []), item.label]
  return result
}, {})

const cargoPositionMap: Record<string, string> = {
  '49仓1号货位': '49仓1号货位',
  '49仓2号货位': '49仓1号货位',
  '49仓-1号货位': '49仓1号货位',
  '49仓-2号货位': '49仓1号货位',
  '49仓-3号货位': '49仓1号货位',
  '49仓-4号货位': '49仓1号货位',
  '50仓1号货位': '50仓1号货位',
  '50仓2号货位': '50仓1号货位',
  '50仓-1号货位': '50仓1号货位',
  '50仓-2号货位': '50仓1号货位',
  '50仓-3号货位': '50仓1号货位',
  '50仓-4号货位': '50仓1号货位',
  '51仓-1号货位': '51仓1号货位',
  '51仓-2号货位': '51仓1号货位',
  '51仓-3号货位': '51仓1号货位',
  '51仓-4号货位': '51仓1号货位',
  '52仓-1号货位': '52仓1号货位',
  '52仓-2号货位': '52仓1号货位',
  '52仓-3号货位': '52仓1号货位',
  '52仓-4号货位': '52仓1号货位',
  '53仓-1号货位': '53仓1号货位',
  '53仓-2号货位': '53仓1号货位',
  '53仓-3号货位': '53仓1号货位',
}

const cargoPositionWarehouseNo = (cargoPosition: string) => cargoPositionOptions.find((item) => item.label === cargoPosition)?.warehouseNo ?? cargoPosition.slice(0, 2)

const normalizeCargoPositionValue = (record: SamplingRecord) => {
  const value = record.cargoPosition?.trim() ?? ''
  if (cargoPositionOptions.some((item) => item.label === value)) return value
  if (cargoPositionMap[value]) return cargoPositionMap[value]
  const availableOptions = cargoPositionByWarehouse[record.warehouseNo] ?? cargoPositionOptions.map((item) => item.label)
  const numericKey = Number((record.sampleNo || record.orderNo).replace(/\D/g, '').slice(-2)) || 1
  return availableOptions[(numericKey - 1) % availableOptions.length]
}

const normalizedCargoPosition = (record: SamplingRecord) => normalizeCargoPositionValue(record)

const displaySurveyNo = (record: SamplingRecord) => record.surveyNo?.trim() || `${record.reason.includes('春季') ? 'CJP' : 'QJP'}-${record.samplingDate.replace(/-/g, '')}-${record.warehouseNo}`

const currentDateTimeText = () => new Date().toLocaleString('zh-CN', { hour12: false }).replace(/\//g, '-')

const code128Patterns = [
  '212222', '222122', '222221', '121223', '121322', '131222', '122213', '122312', '132212', '221213',
  '221312', '231212', '112232', '122132', '122231', '113222', '123122', '123221', '223211', '221132',
  '221231', '213212', '223112', '312131', '311222', '321122', '321221', '312212', '322112', '322211',
  '212123', '212321', '232121', '111323', '131123', '131321', '112313', '132113', '132311', '211313',
  '231113', '231311', '112133', '112331', '132131', '113123', '113321', '133121', '313121', '211331',
  '231131', '213113', '213311', '213131', '311123', '311321', '331121', '312113', '312311', '332111',
  '314111', '221411', '431111', '111224', '111422', '121124', '121421', '141122', '141221', '112214',
  '112412', '122114', '122411', '142112', '142211', '241211', '221114', '413111', '241112', '134111',
  '111242', '121142', '121241', '114212', '124112', '124211', '411212', '421112', '421211', '212141',
  '214121', '412121', '111143', '111341', '131141', '114113', '114311', '411113', '411311', '113141',
  '114131', '311141', '411131', '211412', '211214', '211232', '2331112',
]

const barcodeImageUrl = (value: string) => {
  const text = String(value || '').replace(/[^\x20-\x7E]/g, '')
  const data = text || 'EMPTY'
  const codes = [104, ...data.split('').map((char) => char.charCodeAt(0) - 32)]
  const checksum = codes.reduce((sum, code, index) => sum + code * (index || 1), codes[0]) % 103
  const sequence = [...codes, checksum, 106]
  const moduleWidth = 2
  const height = 72
  const quiet = 18
  let x = quiet
  const rects: string[] = []
  sequence.forEach((code) => {
    code128Patterns[code].split('').forEach((widthText, index) => {
      const width = Number(widthText) * moduleWidth
      if (index % 2 === 0) rects.push(`<rect x="${x}" y="0" width="${width}" height="${height}"/>`)
      x += width
    })
  })
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" width="${x + quiet}" height="${height}" viewBox="0 0 ${x + quiet} ${height}"><rect width="100%" height="100%" fill="white"/><g fill="black">${rects.join('')}</g></svg>`
  return `data:image/svg+xml;charset=utf-8,${encodeURIComponent(svg)}`
}

const qrCodeImageUrl = (value: string) => {
  const text = encodeURIComponent(String(value || ''))
  return `https://api.qrserver.com/v1/create-qr-code/?size=140x140&margin=8&data=${text}`
}

const formatDateText = (date: Date) => `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`

const parseDateText = (value: string) => {
  const [datePart] = value.replace(/\./g, '-').replace(/\//g, '-').split(' ')
  const [year, month, day] = datePart.split('-').map(Number)
  if (!year || !month || !day) return null
  return new Date(year, month - 1, day)
}

const addDaysText = (value: string, days: number) => {
  const date = parseDateText(value)
  if (!date) return '—'
  date.setDate(date.getDate() + days)
  return formatDateText(date)
}

const retainExpireDays = (record: SamplingRecord) => record.retainExpireDays ?? 30

const retainExpireDate = (record: SamplingRecord) => record.retainExpireAt ?? addDaysText(record.retainTime ?? record.handleTime ?? ledgerHandleTime(record), retainExpireDays(record))
const sampleCountWithUnit = (record: Pick<SamplingRecord, 'sampleCount' | 'sampleUnit'>) => record.sampleCount.includes(record.sampleUnit ?? '') ? record.sampleCount : `${record.sampleCount}${record.sampleUnit ?? ''}`
const retainCountWithUnit = (record: Pick<SamplingRecord, 'retainCount' | 'sampleUnit'>) => record.retainCount ? `${record.retainCount}${record.sampleUnit ?? ''}` : '—'

const retainExpireDiffDays = (record: SamplingRecord) => {
  const expireDate = parseDateText(retainExpireDate(record))
  const today = parseDateText(formatDateText(new Date()))
  if (!expireDate || !today) return null
  return Math.ceil((expireDate.getTime() - today.getTime()) / 86400000)
}

const retainExpireProgress = (record: SamplingRecord) => {
  const startDate = parseDateText(record.retainTime ?? record.receiveTime ?? record.samplingDate)
  const expireDate = parseDateText(retainExpireDate(record))
  const today = parseDateText(formatDateText(new Date()))
  if (!startDate || !expireDate || !today) return 0
  const total = expireDate.getTime() - startDate.getTime()
  if (total <= 0) return today.getTime() >= expireDate.getTime() ? 100 : 0
  const elapsed = today.getTime() - startDate.getTime()
  return Math.min(100, Math.max(0, Math.round((elapsed / total) * 100)))
}

const retainStatusLevel = (record: SamplingRecord) => {
  const diffDays = retainExpireDiffDays(record)
  if (diffDays === null) return 'normal'
  if (diffDays < 0) return 'expired'
  if (diffDays <= 30) return 'soon'
  return 'normal'
}

const retainStatusText = (record: SamplingRecord) => {
  const level = retainStatusLevel(record)
  if (level === 'expired') return '已过期'
  if (level === 'soon') return '快过期'
  return '留样'
}

const destroyRecordForRetain = (record: SamplingRecord) => internalDestroyRecords.value.find((destroy) => destroy.destroySourceRetainId === (record.retainRecordId ?? record.sampleNo) || destroy.sampleNo === record.sampleNo)

const retainLifecycleStatus = (record: SamplingRecord) => {
  // 留样主状态只有四种；到期信息通过 retainExpireStatus 单独展示。
  if (record.retainSampleStatus === '已销样') return '已销样'
  const destroyRecord = destroyRecordForRetain(record)
  if (destroyRecord?.destroyApprovalStatus === '已销样') return '已销样'
  if (destroyRecord) return '销样中'
  if (record.retainSampleStatus === '销样中') return '销样中'
  if (!record.retainRecordId) return '留样中'
  if (record.retainApprovalStatus !== '留样完成') return '审批中'
  return '留样中'
}

const retainLifecycleTone = (record: SamplingRecord) => {
  const status = retainLifecycleStatus(record)
  if (status === '已销样') return 'destroyed'
  if (status === '销样中') return 'disposing'
  if (status === '留样中' && retainStatusLevel(record) === 'expired') return 'expired'
  return 'normal'
}

const canStartDestroyFromRetain = (record: SamplingRecord) => {
  return retainLifecycleStatus(record) === '留样中'
    && retainStatusLevel(record) === 'expired'
    && !destroyRecordForRetain(record)
}

const retainWorkflowStatusText = (record: SamplingRecord) => {
  if (!record.retainRecordId) return '历史留样'
  if (record.retainApprovalStatus === '留样完成') return '审批流程完成'
  if (record.retainApprovalStatus === '退回修改') return '待修改'
  if (record.retainApprovalStep) return `${record.retainApprovalStep}审批中`
  return record.retainSampleStatus ?? '未送审'
}

const retainExpireStatus = (record: SamplingRecord) => {
  const diffDays = retainExpireDiffDays(record)
  if (diffDays === null) return '未设置'
  if (diffDays < 0) return `已过期 ${Math.abs(diffDays)} 天`
  if (diffDays === 0) return '今日到期'
  return `剩余 ${diffDays} 天`
}

const openRetainScanDialog = () => {
  retainScanValue.value = ''
  showRetainScanDialog.value = true
}

const readRetainSampleCode = () => {
  const target = allSamplingRecords.value.find((record) => record.sampleNo === retainScanValue.value.trim())
    ?? labelRecords.value.find((record) => !internalRetainRecords.value.some((retain) => retain.sampleNo === record.sampleNo && retain.retainApprovalStatus !== '退回修改'))
  if (target) retainScanValue.value = target.sampleNo
}

const fillRetainFormFromSample = (target: SamplingRecord, existing?: SamplingRecord) => {
  retainForm.value = {
    recordId: existing?.retainRecordId ?? '',
    sampleNo: target.sampleNo,
    sampleName: target.sampleName,
    sampleDate: (target.receiveTime ?? target.samplingDate).slice(0, 10),
    sampleCount: existing?.retainCount ?? target.retainCount ?? target.sampleCount,
    sampleUnit: existing?.sampleUnit ?? target.sampleUnit ?? 'kg',
    specification: existing?.retainSpecification ?? target.sampleCount,
    warehouseNo: existing?.warehouseNo ?? target.warehouseNo,
    cargoPosition: existing?.cargoPosition ?? target.cargoPosition ?? `${target.warehouseNo}仓1号货位`,
    expireAt: existing?.retainExpireAt ?? '',
    sampleStatus: '审批中',
    handlingMethod: existing?.retainHandlingMethod ?? '留存',
    disposalTime: existing?.retainDisposalTime ?? '',
    sourceRecordNo: target.orderNo,
    remark: existing?.remark ?? '',
  }
}

const confirmRetainScan = () => {
  const target = allSamplingRecords.value.find((record) => record.sampleNo === retainScanValue.value.trim())
  if (!target) return
  const existing = internalRetainRecords.value.find((record) => record.sampleNo === target.sampleNo)
  if (existing && existing.retainApprovalStatus !== '退回修改') return
  confirmAction(
    `确认读取样品标签 ${target.sampleNo} 并进入留样登记吗？`,
    () => {
      fillRetainFormFromSample(target, existing)
      showRetainScanDialog.value = false
      showRetainForm.value = true
    },
    { title: '留样标签识别确认', confirmText: '进入留样登记' },
  )
}

const openRetainEditForm = (record: SamplingRecord) => {
  const source = allSamplingRecords.value.find((item) => item.sampleNo === record.sampleNo) ?? record
  fillRetainFormFromSample(source, record)
  showRetainForm.value = true
}

const addRetainApprovalHistory = (record: SamplingRecord, title: string, opinion = '') => {
  const user = currentLoginUser.value
  record.retainApprovalHistory = [
    ...(record.retainApprovalHistory ?? []),
    { title, user: user ? `${user.name}（${user.role}）` : '系统', time: currentDateTimeText(), ...(opinion.trim() ? { opinion: opinion.trim() } : {}) },
  ]
}

const saveRetainForm = () => {
  const form = retainForm.value
  if (!form.sampleNo || !form.sampleName || !form.sampleDate || !form.sampleCount || !form.expireAt) return
  const source = allSamplingRecords.value.find((record) => record.sampleNo === form.sampleNo)
  if (!source) return
  const existing = form.recordId ? internalRetainRecords.value.find((record) => record.retainRecordId === form.recordId) : undefined
  const retainNo = existing?.retainRecordId ?? `LY-${form.sampleNo}-${Date.now()}`
  const actionText = existing ? '重新提交留样登记' : '提交留样登记'
  confirmAction(
    `确认${actionText} ${form.sampleNo} 并进入留样审批吗？`,
    () => {
      const nextRecord: SamplingRecord = {
        ...(existing ?? source),
        orderNo: existing?.orderNo ?? retainNo,
        status: '留样登记',
        labelStatus: '留样',
        approvalStatus: source.approvalStatus,
        approvalStep: undefined,
        reportNo: undefined,
        retainRecordId: retainNo,
        retainCount: form.sampleCount,
        sampleUnit: form.sampleUnit,
        warehouseNo: form.warehouseNo,
        cargoPosition: form.cargoPosition,
        packageType: form.specification,
        retainSpecification: form.specification,
        retainExpireAt: form.expireAt,
        retainSampleStatus: '审批中',
        retainHandlingMethod: form.handlingMethod,
        retainDisposalTime: existing?.retainDisposalTime ?? '',
        retainApprovalStatus: '审批中',
        retainApprovalStep: '审核人',
        retainTime: existing?.retainTime,
        handleTime: currentDateTimeText(),
        remark: form.remark,
        retainApprovalHistory: [
          ...(existing?.retainApprovalHistory ?? []),
          { title: existing ? '重新提交：留样送审' : '留样送审', user: `${currentLoginUser.value?.name ?? '质检员A'}（${currentLoginUser.value?.role ?? '质检员'}）`, time: currentDateTimeText() },
        ],
      }
      if (existing) {
        const index = internalRetainRecords.value.findIndex((record) => record.retainRecordId === retainNo)
        if (index >= 0) internalRetainRecords.value[index] = nextRecord
      } else {
        internalRetainRecords.value.unshift(nextRecord)
      }
      showRetainForm.value = false
    },
    { title: '提交留样审批确认', confirmText: '确认送审' },
  )
}

const selectedRetainRecord = computed(() => internalRetainRecords.value.find((record) => record.retainRecordId === selectedRetainRecordNo.value) ?? internalRetainRecords.value[0])
const retainScanRecord = computed(() => allSamplingRecords.value.find((record) => record.sampleNo === retainScanValue.value.trim()))
const retainScanExistingRecord = computed(() => internalRetainRecords.value.find((record) => record.sampleNo === retainScanValue.value.trim() && record.retainApprovalStatus !== '退回修改'))

const retainApprovalFlowItems = (record?: SamplingRecord) => {
  if (!record) return []
  const history = record.retainApprovalHistory ?? []
  return [
    { name: '留样送审', state: history.some((item) => item.title.includes('留样送审')) || record.retainApprovalStep || record.retainApprovalStatus === '留样完成' ? 'done' : 'todo', icon: Send, desc: '留样登记提交审批' },
    { name: '审核人审批', state: record.retainApprovalStep === '审核人' ? 'active' : history.some((item) => item.title.includes('审核人')) || ['负责人', '批准人'].includes(String(record.retainApprovalStep)) || record.retainApprovalStatus === '留样完成' ? 'done' : 'todo', icon: ClipboardList, desc: '留样信息审核' },
    { name: '负责人审批', state: record.retainApprovalStep === '负责人' ? 'active' : history.some((item) => item.title.includes('负责人')) || record.retainApprovalStep === '批准人' || record.retainApprovalStatus === '留样完成' ? 'done' : 'todo', icon: Edit3, desc: '质量负责人复核' },
    { name: '批准人审批', state: record.retainApprovalStep === '批准人' ? 'active' : history.some((item) => item.title.includes('批准人')) || record.retainApprovalStep === '确认留样' || record.retainApprovalStatus === '留样完成' ? 'done' : 'todo', icon: CheckCircle2, desc: '留样审批批准' },
    { name: '确认留样', state: record.retainApprovalStep === '确认留样' ? 'active' : record.retainApprovalStatus === '留样完成' ? 'done' : 'todo', icon: PackageCheck, desc: '批准人最终确认留样' },
  ]
}

const canApproveRetain = (record?: SamplingRecord) => Boolean(record?.retainApprovalStep && (record.retainApprovalStep === currentLoginUser.value?.role || (record.retainApprovalStep === '确认留样' && currentLoginUser.value?.role === '批准人')))

const openRetainApprovalFlow = (record: SamplingRecord) => {
  selectedRetainRecordNo.value = record.retainRecordId ?? ''
  retainApprovalOpinion.value = ''
  showRetainApprovalFlow.value = true
}

const executeRetainApproval = (record: SamplingRecord, action: 'pass' | 'return', opinion = '') => {
  if (action === 'return') {
    addRetainApprovalHistory(record, `${record.retainApprovalStep}：不同意，退回修改`, opinion)
    record.retainApprovalStatus = '退回修改'
    record.retainSampleStatus = '审批中'
    record.retainApprovalStep = undefined
    record.handleTime = currentDateTimeText()
    return
  }
  if (record.retainApprovalStep === '审核人') {
    addRetainApprovalHistory(record, '审核人：审批通过/同意', opinion)
    record.retainApprovalStep = '负责人'
  } else if (record.retainApprovalStep === '负责人') {
    addRetainApprovalHistory(record, '负责人：审批通过/同意', opinion)
    record.retainApprovalStep = '批准人'
  } else if (record.retainApprovalStep === '批准人') {
    addRetainApprovalHistory(record, '批准人：审批通过/同意', opinion)
    record.retainApprovalStep = '确认留样'
  } else if (record.retainApprovalStep === '确认留样') {
    addRetainApprovalHistory(record, '确认留样：审批流程完成', opinion)
    record.retainApprovalStep = undefined
    record.retainApprovalStatus = '留样完成'
    record.retainSampleStatus = '留样中'
    record.retainTime = record.retainTime ?? currentDateTimeText()
    record.handleTime = currentDateTimeText()
  }
}

const approveRetainRecord = (action: 'pass' | 'return', record?: SamplingRecord) => {
  const target = record ?? selectedRetainRecord.value
  if (!target || !canApproveRetain(target)) return
  if (record) retainApprovalOpinion.value = ''
  if (target.retainRecordId) selectedRetainRecordNo.value = target.retainRecordId
  const opinion = retainApprovalOpinion.value.trim()
  confirmAction(
    action === 'pass' ? target.retainApprovalStep === '确认留样' ? `确认样品 ${target.sampleNo} 正式进入留样并结束流程吗？` : `确认以“${target.retainApprovalStep}”身份通过留样 ${target.sampleNo} 吗？` : `确认退回留样 ${target.sampleNo} 修改吗？`,
    () => {
      executeRetainApproval(target, action, opinion)
      retainApprovalOpinion.value = ''
    },
    { title: action === 'pass' ? target.retainApprovalStep === '确认留样' ? '最终确认留样' : '留样审批通过确认' : '留样退回修改确认', confirmText: action === 'pass' ? target.retainApprovalStep === '确认留样' ? '确认留样' : '确认通过' : '确认退回', danger: action === 'return' },
  )
}

const destroySourceRecords = computed(() => retainRecords.value.filter((record) => {
  const sourceId = record.retainRecordId ?? record.sampleNo
  const editingRecord = internalDestroyRecords.value.find((destroy) => destroy.destroyRecordId === destroyForm.value.recordId)
  const alreadyDestroyed = internalDestroyRecords.value.some((destroy) => destroy.destroySourceRetainId === sourceId && destroy.destroyRecordId !== destroyForm.value.recordId)
  return !alreadyDestroyed
    && record.retainSampleStatus !== '已销样'
    && (retainLifecycleStatus(record) === '留样中' || editingRecord?.destroySourceRetainId === sourceId)
}))

const selectedDestroyRecord = computed(() => internalDestroyRecords.value.find((record) => record.destroyRecordId === selectedDestroyRecordNo.value) ?? internalDestroyRecords.value[0])

const loadDestroySource = () => {
  const source = destroySourceRecords.value.find((record) => (record.retainRecordId ?? record.sampleNo) === destroyForm.value.sourceRetainId)
  if (!source) return
  destroyForm.value = {
    ...destroyForm.value,
    sampleNo: source.sampleNo,
    sampleName: source.sampleName,
    warehouseNo: source.warehouseNo,
    cargoPosition: source.cargoPosition ?? `${source.warehouseNo}仓1号货位`,
    specification: source.retainSpecification ?? source.packageType ?? source.sampleCount,
    variety: source.grainType,
  }
}

const openDestroyForm = (record?: SamplingRecord) => {
  destroyForm.value = { recordId: record?.destroyRecordId ?? '', sourceRetainId: record?.destroySourceRetainId ?? '', sampleNo: record?.sampleNo ?? '', sampleName: record?.sampleName ?? '', warehouseNo: record?.warehouseNo ?? '', cargoPosition: record?.cargoPosition ?? '', specification: record?.retainSpecification ?? record?.packageType ?? '', variety: record?.destroyVariety ?? record?.grainType ?? '', destroySource: record?.destroySource ?? '留样到期', remark: record?.remark ?? '' }
  if (record) loadDestroySource()
  showDestroyForm.value = true
}

const saveDestroyForm = () => {
  const form = destroyForm.value
  const source = destroySourceRecords.value.find((record) => (record.retainRecordId ?? record.sampleNo) === form.sourceRetainId)
    ?? retainRecords.value.find((record) => (record.retainRecordId ?? record.sampleNo) === form.sourceRetainId)
  if (!source || !form.sampleNo || !form.variety || !form.destroySource) return
  const existing = form.recordId ? internalDestroyRecords.value.find((record) => record.destroyRecordId === form.recordId) : undefined
  const destroyNo = existing?.destroyRecordId ?? `XX-${form.sampleNo}-${Date.now()}`
  confirmAction(
    `确认提交样品 ${form.sampleNo} 的销样登记并进入审批吗？`,
    () => {
      const nextRecord: SamplingRecord = {
        ...(existing ?? source),
        orderNo: existing?.orderNo ?? destroyNo,
        status: '销样登记',
        labelStatus: '销样',
        reportNo: undefined,
        destroyRecordId: destroyNo,
        destroySourceRetainId: source.retainRecordId ?? source.sampleNo,
        destroyApprovalStatus: '审批中',
        destroyApprovalStep: '审核人',
        destroyVariety: form.variety,
        destroySource: form.destroySource,
        retainSpecification: form.specification,
        cargoPosition: form.cargoPosition,
        remark: form.remark,
        destroyApprovalHistory: [
          ...(existing?.destroyApprovalHistory ?? []),
          { title: existing ? '重新提交：销样送审' : '销样送审', user: `${currentLoginUser.value?.name ?? '质检员A'}（${currentLoginUser.value?.role ?? '质检员'}）`, time: currentDateTimeText() },
        ],
      }
      if (existing) {
        const index = internalDestroyRecords.value.findIndex((record) => record.destroyRecordId === destroyNo)
        if (index >= 0) internalDestroyRecords.value[index] = nextRecord
      } else {
        internalDestroyRecords.value.unshift(nextRecord)
      }
      source.retainSampleStatus = '销样中'
      source.retainHandlingMethod = form.destroySource
      source.handleTime = currentDateTimeText()
      showDestroyForm.value = false
    },
    { title: '提交销样审批确认', confirmText: '确认送审' },
  )
}

const startDestroyFromRetain = (record: SamplingRecord) => {
  if (!canStartDestroyFromRetain(record)) return
  const destroyNo = `XX-${record.sampleNo}-${Date.now()}`
  const sourceId = record.retainRecordId ?? record.sampleNo
  const sourceText = retainStatusLevel(record) === 'expired' ? '留样到期' : '主动销样'
  confirmAction(
    `样品 ${record.sampleNo} 已过期 ${Math.abs(retainExpireDiffDays(record) ?? 0)} 天，确认销样并自动转入销样审批流程吗？`,
    () => {
      internalDestroyRecords.value.unshift({
        ...record,
        orderNo: destroyNo,
        status: '销样登记',
        labelStatus: '销样',
        reportNo: undefined,
        destroyRecordId: destroyNo,
        destroySourceRetainId: sourceId,
        destroyApprovalStatus: '审批中',
        destroyApprovalStep: '审核人',
        destroyVariety: record.grainType,
        destroySource: sourceText,
        destroyApprovalHistory: [{ title: '销样送审', user: `${currentLoginUser.value?.name ?? '质检员A'}（${currentLoginUser.value?.role ?? '质检员'}）`, time: currentDateTimeText() }],
      })
      record.retainSampleStatus = '销样中'
      record.retainHandlingMethod = sourceText
      record.handleTime = currentDateTimeText()
    },
    { title: '确认销样', confirmText: '确认销样并送审', danger: true },
  )
}

const destroyApprovalFlowItems = (record?: SamplingRecord) => {
  if (!record) return []
  const history = record.destroyApprovalHistory ?? []
  return [
    { name: '销样送审', state: history.some((item) => item.title.includes('销样送审')) || record.destroyApprovalStep || record.destroyApprovalStatus === '已销样' ? 'done' : 'todo', icon: Send, desc: '销样登记提交审批' },
    { name: '审核人审批', state: record.destroyApprovalStep === '审核人' ? 'active' : history.some((item) => item.title.includes('审核人')) || ['负责人', '批准人'].includes(String(record.destroyApprovalStep)) || record.destroyApprovalStatus === '已销样' ? 'done' : 'todo', icon: ClipboardList, desc: '销样信息审核' },
    { name: '负责人审批', state: record.destroyApprovalStep === '负责人' ? 'active' : history.some((item) => item.title.includes('负责人')) || record.destroyApprovalStep === '批准人' || record.destroyApprovalStep === '确认销样' || record.destroyApprovalStatus === '已销样' ? 'done' : 'todo', icon: Edit3, desc: '质量负责人复核' },
    { name: '批准人审批', state: record.destroyApprovalStep === '批准人' ? 'active' : history.some((item) => item.title.includes('批准人')) || record.destroyApprovalStep === '确认销样' || record.destroyApprovalStatus === '已销样' ? 'done' : 'todo', icon: CheckCircle2, desc: '销样审批批准' },
    { name: '确认销样', state: record.destroyApprovalStep === '确认销样' ? 'active' : record.destroyApprovalStatus === '已销样' ? 'done' : 'todo', icon: Trash2, desc: '批准人最终确认销样' },
  ]
}

const canApproveDestroy = (record?: SamplingRecord) => Boolean(record?.destroyApprovalStep && (record.destroyApprovalStep === currentLoginUser.value?.role || (record.destroyApprovalStep === '确认销样' && currentLoginUser.value?.role === '批准人')))

const destroyApprovalTone = (record: SamplingRecord) => {
  if (record.destroyApprovalStatus === '已销样') return 'destroyed'
  if (record.destroyApprovalStatus === '退回修改') return 'returned'
  if (record.destroyApprovalStep === '审核人') return 'review'
  if (record.destroyApprovalStep === '负责人') return 'owner'
  if (record.destroyApprovalStep === '批准人' || record.destroyApprovalStep === '确认销样') return 'approve'
  if (record.destroyApprovalStatus === '审批中') return 'pending'
  return 'history'
}

const openDestroyApprovalFlow = (record: SamplingRecord) => {
  selectedDestroyRecordNo.value = record.destroyRecordId ?? ''
  destroyApprovalOpinion.value = ''
  showDestroyApprovalFlow.value = true
}

const executeDestroyApproval = (record: SamplingRecord, action: 'pass' | 'return', opinion = '') => {
  if (action === 'return') {
    addDestroyApprovalHistory(record, `${record.destroyApprovalStep}：不同意，退回修改`, opinion)
    record.destroyApprovalStatus = '退回修改'
    record.destroyApprovalStep = undefined
    return
  }
  if (record.destroyApprovalStep === '审核人') {
    addDestroyApprovalHistory(record, '审核人：审批通过/同意', opinion)
    record.destroyApprovalStep = '负责人'
  } else if (record.destroyApprovalStep === '负责人') {
    addDestroyApprovalHistory(record, '负责人：审批通过/同意', opinion)
    record.destroyApprovalStep = '批准人'
  } else if (record.destroyApprovalStep === '批准人') {
    addDestroyApprovalHistory(record, '批准人：审批通过/同意', opinion)
    record.destroyApprovalStep = '确认销样'
  } else if (record.destroyApprovalStep === '确认销样') {
    addDestroyApprovalHistory(record, '确认销样：销样流程完成', opinion)
    record.destroyApprovalStep = undefined
    record.destroyApprovalStatus = '已销样'
    record.destroyTime = currentDateTimeText()
    record.handleTime = record.destroyTime
    const source = internalRetainRecords.value.find((retain) => retain.retainRecordId === record.destroySourceRetainId || retain.sampleNo === record.sampleNo)
      ?? allSamplingRecords.value.find((retain) => retain.retainRecordId === record.destroySourceRetainId || retain.sampleNo === record.sampleNo)
    if (source) {
      source.retainSampleStatus = '已销样'
      source.retainDisposalTime = record.destroyTime
    }
  }
}

const addDestroyApprovalHistory = (record: SamplingRecord, title: string, opinion = '') => {
  const user = currentLoginUser.value
  record.destroyApprovalHistory = [
    ...(record.destroyApprovalHistory ?? []),
    { title, user: user ? `${user.name}（${user.role}）` : '系统', time: currentDateTimeText(), ...(opinion.trim() ? { opinion: opinion.trim() } : {}) },
  ]
}

const approveDestroyRecord = (action: 'pass' | 'return', record?: SamplingRecord) => {
  const target = record ?? selectedDestroyRecord.value
  if (!target || !canApproveDestroy(target)) return
  if (record) destroyApprovalOpinion.value = ''
  if (target.destroyRecordId) selectedDestroyRecordNo.value = target.destroyRecordId
  const opinion = destroyApprovalOpinion.value.trim()
  confirmAction(
    action === 'pass' ? target.destroyApprovalStep === '确认销样' ? `确认样品 ${target.sampleNo} 正式销样并结束流程吗？` : `确认以“${target.destroyApprovalStep}”身份通过样品 ${target.sampleNo} 的销样申请吗？` : `确认退回样品 ${target.sampleNo} 的销样申请修改吗？`,
    () => {
      executeDestroyApproval(target, action, opinion)
      destroyApprovalOpinion.value = ''
    },
    { title: action === 'pass' ? target.destroyApprovalStep === '确认销样' ? '最终确认销样' : '销样审批通过确认' : '销样退回修改确认', confirmText: action === 'pass' ? target.destroyApprovalStep === '确认销样' ? '确认销样' : '确认通过' : '确认退回', danger: action === 'return' },
  )
}

const reportConclusion = (record: SamplingRecord) => {
  const risk = qualityRiskLevel(record)
  if (risk.code === 'normal') return '质量合格'
  return risk.label
}

const qualityGradeText = (record: SamplingRecord) => {
  if (record.reportMeta?.qualityGrade) return record.reportMeta.qualityGrade
  return reportHasUnqualified(record) ? '不合格' : '一等'
}

const qualifiedText = (record: SamplingRecord): '是' | '否' => {
  if (record.reportMeta?.qualified === '是' || record.reportMeta?.qualified === '否') return record.reportMeta.qualified
  return reportHasUnqualified(record) ? '否' : '是'
}

const qualityGradeTone = (record: SamplingRecord) => {
  const grade = qualityGradeText(record)
  if (qualifiedText(record) === '否' || grade.includes('不合格')) return 'fail'
  if (grade.includes('三等') || grade.includes('等外')) return 'warning'
  return 'pass'
}

const reportQualityGrade = (record: SamplingRecord) => {
  const risk = qualityRiskLevel(record)
  if (risk.code === 'level1') return '不合格（一级预警）'
  if (risk.code === 'level2') return '重点复核（二级预警）'
  if (risk.code === 'level3') return '待复核（三级预警）'
  return '一等（质量正常）'
}

const reportConclusionTone = (record: SamplingRecord) => {
  const code = qualityRiskLevel(record).code
  if (code === 'normal') return 'ok'
  if (code === 'level1') return 'alarm'
  return 'warning'
}

const reportMeasureValue = (record: SamplingRecord, items: string[]) => {
  const matchedItem = items.find((item) => record.reportResults?.[item] || activeQualityStandards(record).some((standard) => standard.item === item))
  if (!matchedItem) return '—'
  const detectValue = qualityDetectValue(record, matchedItem)
  return detectValue || '—'
}

const reportColorOdorValue = (record: SamplingRecord) => {
  const color = qualityResultValue(record, '色泽')
  const odor = qualityResultValue(record, '气味')
  if (color === '合格' && odor === '合格') return '基本正常'
  if (color === '不合格' || odor === '不合格') return `色泽${color}，气味${odor}`
  return '—'
}

const cargoDetailRows = computed(() => completedReportRecords.value.map((record) => ({
  ...record,
  position: normalizedCargoPosition(record),
  warehouseNo: cargoPositionWarehouseNo(normalizedCargoPosition(record)),
  stock: warehouseMarkers.value.find((item) => item.code === `${cargoPositionWarehouseNo(normalizedCargoPosition(record))}仓`)?.stockQuantity ?? '—',
  conclusion: reportConclusion(record),
  grade: reportQualityGrade(record),
  unqualified: reportUnqualifiedItems(record),
})))

const warehouseReportGrainTabs = ['玉米', '大豆'] as const

const warehouseReportIndicators: Record<typeof warehouseReportGrainTabs[number], { item: string; label: string }[]> = {
  玉米: [
    { item: '水分 (g/100g)', label: '水分（g/100g）' },
    { item: '杂质 (%)', label: '杂质（%）' },
    { item: '容重 (g/L)', label: '容重（g/L）' },
    { item: '不完善粒总量 (%)', label: '不完善粒总量（%）' },
    { item: '黄粒米 (%)', label: '黄粒米（%）' },
    { item: '脂肪酸值 (mgKOH/100g)', label: '脂肪酸值（mgKOH/100g）' },
    { item: '黄曲霉毒素B1 (μg/kg)', label: '黄曲霉毒素B1（μg/kg）' },
    { item: '色泽、气味', label: '色泽、气味' },
  ],
  大豆: [
    { item: '水分 (g/100g)', label: '水分（g/100g）' },
    { item: '杂质 (%)', label: '杂质（%）' },
    { item: '完整粒率 (%)', label: '完整粒率（%）' },
    { item: '损伤粒率 (%)', label: '损伤粒率（%）' },
    { item: '热损伤粒率 (%)', label: '热损伤粒率（%）' },
    { item: '破碎粒率 (%)', label: '破碎粒率（%）' },
    { item: '霉变粒率 (%)', label: '霉变粒率（%）' },
    { item: '粗脂肪酸值 (mg/g)', label: '粗脂肪酸值（mg/g）' },
    { item: '蛋白质含量 (g/100g)', label: '蛋白质含量（g/100g）' },
    { item: '脂肪含量 (g/100g)', label: '脂肪含量（g/100g）' },
    { item: '色泽、气味', label: '色泽、气味' },
  ],
}

const activeWarehouseReportIndicators = computed(() => warehouseReportIndicators[activeWarehouseReportGrain.value])

const warehouseReportIndicatorStandard = (item: string) => {
  if (item === '色泽、气味') return '正常'
  const standards = qualityStandards[activeWarehouseReportGrain.value]
  return Object.values(standards).flat().find((standard) => standard.item === item)?.standard ?? '—'
}

const warehouseReportIndicatorValue = (record: SamplingRecord, item: string) => {
  if (item === '色泽、气味') {
    const color = qualityDetectValue(record, '色泽') || qualityResultValue(record, '色泽')
    const odor = qualityDetectValue(record, '气味') || qualityResultValue(record, '气味')
    return color && odor ? `${color}/${odor}` : '—'
  }
  return reportMeasureValue(record, [item])
}

const warehouseReportRows = computed(() => completedReportRecords.value
  .filter((record) => record.grainType === activeWarehouseReportGrain.value)
  .map((record) => ({
  record,
  position: normalizedCargoPosition(record),
  warehouseNo: cargoPositionWarehouseNo(normalizedCargoPosition(record)),
  quantity: record.representativeQuantity ?? '100.000',
  grade: reportQualityGrade(record),
  storageDate: record.storageDate ?? record.samplingDate,
  origin: record.origin ?? '江苏',
  productionYear: record.productionYear ?? record.samplingDate.slice(0, 4),
  packageType: record.packageType ?? '散装',
  detectDate: (record.receiveTime ?? record.samplingDate).slice(0, 10),
  moisture: reportMeasureValue(record, ['水分 (g/100g)', '水分及挥发物(%)']),
  impurity: reportMeasureValue(record, ['杂质 (%)']),
  completeRate: reportMeasureValue(record, ['完整粒率 (%)', '容重 (g/L)']),
  damageRate: reportMeasureValue(record, ['破碎粒率 (%)', '不完善粒总量 (%)']),
  heatDamageRate: reportMeasureValue(record, ['热损伤粒率 (%)']),
  proteinRatio: reportMeasureValue(record, ['蛋白质含量 (g/100g)', '品尝评分值 (分)']),
  crudeFattyAcid: reportMeasureValue(record, ['粗脂肪酸值 (mg/g)', '脂肪酸值 (mgKOH/100g)', '酸价(KOH)/(mg/g)']),
  fatContent: reportMeasureValue(record, ['脂肪含量 (g/100g)']),
  colorOdor: reportColorOdorValue(record),
  moldRate: reportMeasureValue(record, ['霉变粒率 (%)']),
  agency: '镇江直属库检化验室',
  reportNo: record.reportNo ?? '待生成',
  recorder: record.reportMeta?.compiler ?? '系统记录员',
  purpose: record.reason,
  remark: reportConclusion(record),
})))

const warehouseReportMeta = computed(() => {
  const rows = warehouseReportRows.value
  const uniq = (values: string[]) => [...new Set(values.filter(Boolean))]
  const totalQuantity = rows.reduce((total, row) => total + (Number(row.quantity) || 0), 0)
  const hasAlarm = rows.some((row) => qualityRiskLevel(row.record).code === 'level1')
  const hasWarning = rows.some((row) => ['level2', 'level3'].includes(qualityRiskLevel(row.record).code))
  return {
    enterprise: '中央储备粮镇江直属库有限公司',
    nature: 'GYC',
    positions: uniq(rows.map((row) => row.position)).join('、') || '—',
    grainTypes: uniq(rows.map((row) => row.record.grainType)).join('、') || '—',
    totalQuantity: rows.length ? totalQuantity.toFixed(3) : '—',
    grade: hasAlarm ? '不合格' : hasWarning ? '待复核' : '一等',
    storageDate: uniq(rows.map((row) => row.storageDate)).slice(0, 2).join('、') || '—',
    origin: uniq(rows.map((row) => row.origin)).join('、') || '—',
    productionYear: uniq(rows.map((row) => row.productionYear)).join('、') || '—',
    packageType: uniq(rows.map((row) => row.packageType)).join('、') || '—',
  }
})

const warehouseReportBlankRows = computed(() => Array.from({ length: Math.max(0, 6 - warehouseReportRows.value.length) }))
const warehouseReportColumnCount = computed(() => activeWarehouseReportIndicators.value.length + 7)
const cargoReportBlankRows = computed(() => Array.from({ length: Math.max(0, 6 - cargoDetailRows.value.length) }))
const acceptanceReportBlankRows = computed(() => Array.from({ length: Math.max(0, 6 - acceptanceRows.value.length) }))
const seasonReportBlankRows = computed(() => Array.from({ length: Math.max(0, 6 - seasonRows.value.length) }))

const warehouseSummaryRows = computed(() => {
  const positions = cargoPositionOptions.map((item) => item.label)
  return positions.map((position) => {
  const records = completedReportRecords.value.filter((record) => normalizedCargoPosition(record) === position)
  const warningCount = records.filter((record) => ['level2', 'level3'].includes(qualityRiskLevel(record).code)).length
  const alarmCount = records.filter((record) => qualityRiskLevel(record).code === 'level1').length
  const passCount = records.filter((record) => qualityRiskLevel(record).code === 'normal').length
  return {
    warehouseNo: cargoPositionWarehouseNo(position),
    position,
    sampleCount: records.length,
    passCount,
    warningCount,
    alarmCount,
    passRate: records.length ? `${Math.round((passCount / records.length) * 100)}%` : '—',
    mainIssue: records.flatMap((record) => reportUnqualifiedItems(record))[0] ?? '无',
  }
})
})

const acceptanceRows = computed(() => completedReportRecords.value.filter((record) => record.reason.includes('入库')))
const seasonRows = computed(() => completedReportRecords.value.filter((record) => record.reason.includes('春季') || record.reason.includes('秋季')))

const labInstruments = ref<LabInstrument[]>([
  { code: 'YQ-CRQ-001', name: '谷物容重器', model: 'GHCS-1000', location: '检化验室A区', custodian: '质检员A', status: '在用', calibration: '2026-08-15', maintenance: '2026-07-10', lastMaintenance: '2026-06-10', maintenanceType: '月度维护', maintenanceHandler: '', usage: 86, type: '物理检测设备', lastCalibration: '2025-08-15' },
  { code: 'YQ-YD-002', name: '小麦硬度指数测定仪', model: 'JYDB-100', location: '物理检测区', custodian: '王帅', status: '在用', calibration: '2026-09-02', maintenance: '2026-07-12', lastMaintenance: '2026-06-12', maintenanceType: '清洁校验', maintenanceHandler: '', usage: 72, type: '物理检测设备', lastCalibration: '2025-09-02' },
  { code: 'YQ-SF-003', name: '谷物水分测定仪', model: 'LDS-1G', location: '快速检测区', custodian: '质检员A', status: '在用', calibration: '2026-07-18', maintenance: '2026-07-08', lastMaintenance: '2026-06-08', maintenanceType: '月度维护', maintenanceHandler: '', usage: 91, type: '水分检测设备', lastCalibration: '2025-07-18' },
  { code: 'YQ-ZZ-004', name: '杂质筛选器', model: 'JJSG22×12', location: '样品制备区', custodian: '李审核', status: '待保养', calibration: '2026-10-20', maintenance: '2026-07-05', lastMaintenance: '2026-06-05', maintenanceType: '清洁保养', maintenanceHandler: '', usage: 58, type: '样品制备设备', lastCalibration: '2025-10-20' },
  { code: 'YQ-ZFS-005', name: '脂肪酸值测定仪', model: 'JZSG-II', location: '化学检测区', custodian: '赵批准', status: '检定临期', calibration: '2026-07-14', maintenance: '2026-07-14', lastMaintenance: '2026-06-14', maintenanceType: '月度维护', maintenanceHandler: '', usage: 64, type: '化学检测设备', lastCalibration: '2025-07-14' },
  { code: 'YQ-CS-006', name: '粮食测水仪', model: 'PM-8188', location: '现场快检区', custodian: '质检员A', status: '在用', calibration: '2026-11-12', maintenance: '2026-07-16', lastMaintenance: '2026-06-16', maintenanceType: '功能校验', maintenanceHandler: '', usage: 79, type: '水分检测设备', lastCalibration: '2025-11-12' },
  { code: 'YQ-HW-007', name: '恒温培养箱', model: 'DHP-9052', location: '化学检测区', custodian: '赵批准', status: '在用', calibration: '2026-12-05', maintenance: '2026-08-01', lastMaintenance: '2026-07-01', maintenanceType: '月度维护', maintenanceHandler: '赵批准', usage: 45, type: '环境控制设备', lastCalibration: '2025-12-05' },
])

const labInstrumentCategories = ref<LabInstrumentCategory[]>([
  { type: '物理检测设备', description: '谷物物性与容重检测', storageArea: '检化验室A区', note: '常规物理指标' },
  { type: '水分检测设备', description: '水分及挥发物检测', storageArea: '快速检测区', note: '高频快检' },
  { type: '样品制备设备', description: '样品筛选与前处理', storageArea: '样品制备区', note: '样品前处理' },
  { type: '化学检测设备', description: '理化指标与脂肪酸值检测', storageArea: '化学检测区', note: '理化分析' },
  { type: '环境控制设备', description: '恒温与环境控制', storageArea: '化学检测区', note: '环境保持' },
])
selectedCalibrationInstrumentCode.value = labInstruments.value[0]?.code ?? ''

const buildCalibrationHistory = () => {
  const rows: LabCalibrationHistoryRecord[] = []

  labInstruments.value.forEach((instrument, index) => {
    const planYear = Number(instrument.calibration.slice(0, 4))
    const monthDay = instrument.calibration.slice(5)
    const recentYear = Math.max(planYear - 1, 2024)
    const olderYear = Math.max(recentYear - 1, 2023)
    const recentAgency = index % 2 === 0 ? '镇江市计量测试中心' : '江苏粮油质检站'
    const olderAgency = index % 2 === 0 ? '江苏粮油质检站' : '镇江市计量测试中心'

    rows.push({
      recordNo: `${instrument.code}-H1`,
      instrumentCode: instrument.code,
      instrumentName: instrument.name,
      actualDate: `${olderYear}-${monthDay}`,
      planDate: `${olderYear}-${monthDay}`,
      agency: olderAgency,
      certificate: `JD${olderYear}${String(index + 1).padStart(3, '0')}`,
      result: '合格',
    })

    rows.push({
      recordNo: `${instrument.code}-H2`,
      instrumentCode: instrument.code,
      instrumentName: instrument.name,
      actualDate: instrument.lastCalibration,
      planDate: instrument.lastCalibration,
      agency: recentAgency,
      certificate: `JD${recentYear}${String(index + 11).padStart(3, '0')}`,
      result: '合格',
    })
  })

  return rows.sort((a, b) => b.actualDate.localeCompare(a.actualDate))
}

const labCalibrationHistory = ref<LabCalibrationHistoryRecord[]>(buildCalibrationHistory())

const environmentRanges: { key: EnvironmentRange; label: string; points: number }[] = [
  { key: 'day', label: '当日', points: 8 },
  { key: 'week', label: '近一周', points: 7 },
  { key: 'month', label: '近一个月', points: 30 },
  { key: 'halfYear', label: '近半年', points: 6 },
]
const activeEnvironmentRangeLabel = computed(() => environmentRanges.find((item) => item.key === activeEnvironmentRange.value)?.label ?? '当日')

const environmentAlarmTypes: { key: EnvironmentAlarmType; label: string }[] = [
  { key: 'all', label: '全部状态' },
  { key: 'normal', label: '正常' },
  { key: 'temperature', label: '温度报警' },
  { key: 'humidity', label: '湿度报警' },
  { key: 'both', label: '温湿度报警' },
]

const environmentAreas = ['检化验室A区', '物理检测区', '化学检测区', '快速检测区', '样品制备区']

const environmentStatus = (temperature: number, humidity: number) => {
  if (temperature > 30 && humidity > 60) return '温湿度报警'
  if (temperature > 30) return '温度报警'
  if (humidity > 60) return '湿度报警'
  return '正常'
}

const localDateText = (date: Date) => {
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${date.getFullYear()}-${month}-${day}`
}

const buildEnvironmentHistory = (): EnvironmentRecord[] => {
  const rows: EnvironmentRecord[] = []
  const sampleTimes = ['08:00', '12:00', '16:00', '20:00']
  const endDate = new Date(`${formatDateText(new Date())}T00:00:00`)
  const alarmSamples = new Map<string, { temperature: number; humidity: number }>([
    ['0-0-1', { temperature: 31.2, humidity: 55 }],
    ['0-1-2', { temperature: 26.8, humidity: 62 }],
    ['0-2-3', { temperature: 31.6, humidity: 64 }],
    ['3-3-1', { temperature: 31.1, humidity: 56 }],
    ['5-4-2', { temperature: 27.1, humidity: 63 }],
    ['6-2-3', { temperature: 31.3, humidity: 65 }],
    ['18-0-3', { temperature: 31.5, humidity: 55 }],
    ['24-3-2', { temperature: 26.4, humidity: 63 }],
    ['29-1-1', { temperature: 31.7, humidity: 64 }],
    ['57-2-1', { temperature: 26.4, humidity: 63 }],
    ['96-4-2', { temperature: 31.8, humidity: 64 }],
    ['142-0-3', { temperature: 31.4, humidity: 56 }],
  ])

  for (let offset = 0; offset < 180; offset += 1) {
    const current = new Date(endDate)
    current.setDate(endDate.getDate() - offset)
    const dateText = localDateText(current)

    environmentAreas.forEach((area, areaIndex) => {
      sampleTimes.forEach((time, timeIndex) => {
        const seasonal = Math.sin((180 - offset) / 18) * 1.8
        const daily = [0, 0.7, 1.1, 0.4][timeIndex]
        const areaHeat = areaIndex * 0.45
        let temperature = Number((24.1 + seasonal + daily + areaHeat).toFixed(1))
        let humidity = Math.round(48 + Math.cos(offset / 11) * 3 + areaIndex * 1.1 + timeIndex * 1.2)
        const alarmSample = alarmSamples.get(`${offset}-${areaIndex}-${timeIndex}`)

        if (alarmSample) {
          temperature = alarmSample.temperature
          humidity = alarmSample.humidity
        }

        rows.push({
          time: `${dateText} ${time}`,
          area,
          temperature,
          humidity,
          status: environmentStatus(temperature, humidity),
        })
      })
    })
  }

  return rows.sort((a, b) => a.time.localeCompare(b.time))
}

const environmentRecords = buildEnvironmentHistory()
const environmentRealtimeCollectedAt = ref(currentDateTimeText())
const realtimeEnvironmentRecords = computed<EnvironmentRecord[]>(() => [
  { time: environmentRealtimeCollectedAt.value, area: '检化验室A区', temperature: 25.4, humidity: 56, status: environmentStatus(25.4, 56) },
  { time: environmentRealtimeCollectedAt.value, area: '物理检测区', temperature: 25.9, humidity: 57, status: environmentStatus(25.9, 57) },
  { time: environmentRealtimeCollectedAt.value, area: '化学检测区', temperature: 26.2, humidity: 58, status: environmentStatus(26.2, 58) },
  { time: environmentRealtimeCollectedAt.value, area: '快速检测区', temperature: 25.7, humidity: 56, status: environmentStatus(25.7, 56) },
  { time: environmentRealtimeCollectedAt.value, area: '样品制备区', temperature: 26.1, humidity: 59, status: environmentStatus(26.1, 59) },
])
const environmentRealtimeTime = computed(() => realtimeEnvironmentRecords.value[0]?.time ?? '—')
const environmentChartSourceRecords = computed(() => {
  const startTime = environmentRangeStartMap[activeEnvironmentRange.value]
  return environmentRecords
    .filter((item) => item.time.slice(0, 10) >= startTime)
    .filter((item) => activeEnvironmentArea.value === '全部区域' || item.area === activeEnvironmentArea.value)
    .sort((a, b) => a.time.localeCompare(b.time))
})
const environmentRangeRecords = computed(() => {
  const startTime = environmentRangeStartMap[activeEnvironmentRange.value]
  return environmentRecords
    .filter((item) => item.time.slice(0, 10) >= startTime)
    .filter((item) => activeEnvironmentArea.value === '全部区域' || item.area === activeEnvironmentArea.value)
    .sort((a, b) => a.time.localeCompare(b.time))
})

const normalizeLabInstrumentType = (type: string) => type.trim()

const instrumentTypeOptions = computed(() => {
  const categoryTypes = labInstrumentCategories.value.map((item) => item.type)
  const fallbackTypes = labInstruments.value.map((item) => item.type)
  return [...new Set([...categoryTypes, ...fallbackTypes])]
})

const labStats = computed(() => [
  { label: '仪器总数', value: labInstruments.value.length, tone: 'blue' },
  { label: '设备类型', value: labInstrumentCategories.value.length, tone: 'green' },
  { label: '待检定/保养', value: labInstruments.value.filter((item) => calibrationStatus(item).tone !== 'ok' || item.status === '待保养').length, tone: 'orange' },
  { label: '环境异常', value: filteredEnvironmentRecords.value.filter((item) => item.status !== '正常').length, tone: 'red' },
])

const calibrationPlanRows = computed(() => labInstruments.value.map((item, index) => {
  const status = calibrationStatus(item)
  return {
    instrument: item.name,
    code: item.code,
    type: item.type,
    planDate: item.calibration,
    agency: index % 2 === 0 ? '镇江市计量测试中心' : '江苏粮油质检站',
    lastDate: item.lastCalibration,
    remainingDays: calibrationRemainingDays(item),
    progress: calibrationProgress(item),
    statusLabel: status.label,
    statusTone: status.tone,
    certificate: status.tone === 'ok' ? `JD2026${String(index + 1).padStart(3, '0')}` : '待上传',
  }
}))

const selectedCalibrationInstrument = computed(() => {
  return labInstruments.value.find((item) => item.code === selectedCalibrationInstrumentCode.value) ?? labInstruments.value[0] ?? null
})

const selectedCalibrationHistoryRows = computed(() => {
  const instrument = selectedCalibrationInstrument.value
  if (!instrument) return []
  const status = calibrationStatus(instrument)
  const historyRows = labCalibrationHistory.value
    .filter((item) => item.instrumentCode === instrument.code)
    .sort((a, b) => b.actualDate.localeCompare(a.actualDate))

  return [
    {
      recordNo: `${instrument.code}-CURRENT`,
      instrumentCode: instrument.code,
      instrumentName: instrument.name,
      actualDate: '待执行',
      planDate: instrument.calibration,
      agency: '当前检定计划',
      certificate: '待生成',
      result: status.tone === 'danger' ? '已过期' : status.tone === 'warn' ? '临期' : '待检定',
    },
    ...historyRows,
  ]
})

const openCalibrationHistory = (code: string) => {
  selectedCalibrationInstrumentCode.value = code
  showCalibrationHistoryDialog.value = true
}

const calibrationStats = computed(() => [
  { label: '仪器总数', value: calibrationPlanRows.value.length, tone: 'blue' },
  { label: '临期设备', value: calibrationPlanRows.value.filter((item) => item.statusTone === 'warn').length, tone: 'orange' },
  { label: '过期设备', value: calibrationPlanRows.value.filter((item) => item.statusTone === 'danger').length, tone: 'red' },
  { label: '历史记录', value: labCalibrationHistory.value.length, tone: 'green' },
])

const maintenanceRecords = computed(() => labInstruments.value.map((item) => {
  const status = maintenanceStatus(item)
  const isAbandoned = item.status === '弃用'
  return {
    instrument: item.name,
    code: item.code,
    lastDate: item.lastMaintenance,
    nextDate: item.maintenance,
    type: item.maintenanceType,
    handler: item.status === '弃用' ? item.abandonHandler || '未登记' : item.maintenanceHandler || '未登记',
    remainingDays: maintenanceRemainingDays(item),
    progress: maintenanceProgress(item),
    statusLabel: status.label,
    statusTone: status.tone,
    isAbandoned,
    canMaintain: !isAbandoned && (status.tone === 'warn' || status.tone === 'danger'),
  }
}))

const labInstrumentTypeSummary = computed(() => {
  const summary = labInstrumentCategories.value.map((category) => {
    const rows = labInstruments.value.filter((instrument) => instrument.type === category.type)
    return {
      type: category.type,
      description: category.description,
      storageArea: category.storageArea,
      note: category.note,
      count: rows.length,
      running: rows.filter((instrument) => instrument.status === '在用').length,
      warning: rows.filter((instrument) => calibrationStatus(instrument).tone !== 'ok' || instrument.status !== '在用').length,
    }
  })

  const extraTypes = labInstruments.value
    .map((instrument) => instrument.type)
    .filter((type) => !labInstrumentCategories.value.some((category) => category.type === type))

  extraTypes.forEach((type) => {
    const rows = labInstruments.value.filter((instrument) => instrument.type === type)
    summary.push({
      type,
      description: '未配置类别说明',
      storageArea: '未指定',
      note: '补充类别',
      count: rows.length,
      running: rows.filter((instrument) => instrument.status === '在用').length,
      warning: rows.filter((instrument) => calibrationStatus(instrument).tone !== 'ok' || instrument.status !== '在用').length,
    })
  })

  return summary
})

const instrumentStatusTone = (instrument: LabInstrument) => {
  if (instrument.status === '弃用' || calibrationStatus(instrument).tone === 'danger') return 'danger'
  if (instrument.status !== '在用' || calibrationStatus(instrument).tone === 'warn') return 'warn'
  return 'ok'
}

const instrumentTypeIcon = (type: string) => {
  if (type.includes('水分')) return Droplets
  if (type.includes('化学')) return FlaskConical
  if (type.includes('制备')) return Settings2
  if (type.includes('环境')) return ThermometerSun
  return Scale
}

const instrumentTypeTone = (type: string) => {
  if (type.includes('水分')) return 'moisture'
  if (type.includes('化学')) return 'chemistry'
  if (type.includes('制备')) return 'preparation'
  if (type.includes('环境')) return 'environment'
  return 'physical'
}

const activeInstrumentTypeRows = computed(() => labInstruments.value.filter((item) => item.type === activeInstrumentType.value))

watch(
  () => instrumentTypeOptions.value,
  (types) => {
    if (!types.length) return
    if (!types.includes(activeInstrumentType.value)) {
      activeInstrumentType.value = types[0]
    }
  },
  { immediate: true },
)

watch(
  () => labInstruments.value.map((item) => item.code),
  (codes) => {
    if (!codes.length) return
    if (!codes.includes(selectedCalibrationInstrumentCode.value)) {
      selectedCalibrationInstrumentCode.value = codes[0]
    }
  },
  { immediate: true },
)

const dayDiff = (from: string | Date, to: string | Date) => {
  const start = typeof from === 'string' ? new Date(`${from.slice(0, 10)}T00:00:00`) : from
  const end = typeof to === 'string' ? new Date(`${to.slice(0, 10)}T00:00:00`) : to
  return Math.ceil((end.getTime() - start.getTime()) / 86400000)
}

const clampPercent = (value: number) => Math.max(0, Math.min(100, Math.round(value)))

const calibrationProgress = (instrument: LabInstrument) => {
  const totalDays = Math.max(1, dayDiff(instrument.lastCalibration, instrument.calibration))
  const elapsedDays = Math.max(0, dayDiff(instrument.lastCalibration, '2026-07-12'))
  return clampPercent((elapsedDays / totalDays) * 100)
}

const calibrationRemainingDays = (instrument: LabInstrument) => dayDiff('2026-07-12', instrument.calibration)

const calibrationStatus = (instrument: LabInstrument) => {
  const remainingDays = calibrationRemainingDays(instrument)
  if (remainingDays < 0) return { label: `已过期 ${Math.abs(remainingDays)} 天`, tone: 'danger' }
  if (remainingDays <= 7) return { label: `${remainingDays} 天后到期`, tone: 'warn' }
  return { label: `${remainingDays} 天后到期`, tone: 'ok' }
}

const formatEnvironmentLabel = (range: EnvironmentRange, value: string) => {
  if (range === 'day') return value.slice(11, 16)
  if (range === 'halfYear') return value.slice(0, 7).replace('-', '/')
  return value.slice(5, 10).replace('-', '/')
}

const formatEnvironmentTooltipDate = (range: EnvironmentRange, value: string) => {
  if (range === 'day') return value.slice(0, 16)
  if (range === 'halfYear') return value.slice(0, 7).replace('-', '/')
  return value.slice(0, 10)
}

const updateInstrumentCalibration = (code: string, value: string) => {
  const target = labInstruments.value.find((item) => item.code === code)
  if (target) {
    target.calibration = value
    const status = calibrationStatus(target)
    if (target.status !== '维护保养中') {
      target.status = status.tone === 'warn' ? '检定临期' : status.tone === 'danger' ? '检定过期' : '在用'
    }
  }
}

const maintenanceProgress = (instrument: LabInstrument) => {
  const totalDays = Math.max(1, dayDiff(instrument.lastMaintenance, instrument.maintenance))
  const elapsedDays = Math.max(0, dayDiff(instrument.lastMaintenance, '2026-07-12'))
  return clampPercent((elapsedDays / totalDays) * 100)
}

const maintenanceRemainingDays = (instrument: LabInstrument) => dayDiff('2026-07-12', instrument.maintenance)

const maintenanceStatus = (instrument: LabInstrument) => {
  if (instrument.status === '弃用') return { label: '弃用', tone: 'danger' }
  const remainingDays = maintenanceRemainingDays(instrument)
  if (remainingDays < 0) return { label: '已逾期', tone: 'danger' }
  if (remainingDays <= 10) return { label: '待维护', tone: 'warn' }
  return { label: '维护保养中', tone: 'info' }
}

const openMaintenanceForm = (instrument: LabInstrument) => {
  selectedMaintenanceInstrumentCode.value = instrument.code
  maintenanceForm.value = {
    date: '2026-07-12',
    type: instrument.maintenanceType,
    handler: instrument.maintenanceHandler || instrument.custodian,
    nextDate: addDaysToDateText('2026-07-12', 30),
  }
  showMaintenanceForm.value = true
}

const saveMaintenanceRecord = () => {
  const target = labInstruments.value.find((item) => item.code === selectedMaintenanceInstrumentCode.value)
  if (!target) return

  target.lastMaintenance = maintenanceForm.value.date || '2026-07-12'
  target.maintenance = maintenanceForm.value.nextDate || addDaysToDateText(target.lastMaintenance, 30)
  target.maintenanceType = maintenanceForm.value.type || '月度维护'
  target.maintenanceHandler = maintenanceForm.value.handler || target.custodian
  target.status = '维护保养中'
  showMaintenanceForm.value = false
}

const openAbandonForm = (instrument: LabInstrument) => {
  selectedAbandonInstrumentCode.value = instrument.code
  abandonForm.value = {
    date: '2026-07-12',
    handler: instrument.abandonHandler || instrument.custodian,
  }
  showAbandonForm.value = true
}

const saveAbandonRecord = () => {
  const target = labInstruments.value.find((item) => item.code === selectedAbandonInstrumentCode.value)
  if (!target) return

  target.abandonedAt = abandonForm.value.date || '2026-07-12'
  target.abandonHandler = abandonForm.value.handler || target.custodian
  target.status = '弃用'
  showAbandonForm.value = false
}

const environmentRangeStartMap: Record<EnvironmentRange, string> = {
  day: formatDateText(new Date()),
  week: addDaysText(formatDateText(new Date()), -6),
  month: addDaysText(formatDateText(new Date()), -29),
  halfYear: addDaysText(formatDateText(new Date()), -179),
}

const filteredEnvironmentRecords = computed(() => {
  return environmentRangeRecords.value.filter((item) => {
    if (activeEnvironmentAlarmType.value === 'all') return true
    if (activeEnvironmentAlarmType.value === 'normal') return item.status === '正常'
    if (activeEnvironmentAlarmType.value === 'temperature') return item.status === '温度报警'
    if (activeEnvironmentAlarmType.value === 'humidity') return item.status === '湿度报警'
    return item.status === '温湿度报警'
  })
    .sort((a, b) => a.time.localeCompare(b.time))
})

const environmentDisplayRecords = computed(() => {
  const rows = filteredEnvironmentRecords.value
  if (activeEnvironmentRange.value === 'day') return [...rows].reverse()

  const groupedRows = new Map<string, EnvironmentRecord>()
  rows.forEach((row) => {
    const key = activeEnvironmentRange.value === 'halfYear'
      ? `${row.time.slice(0, 7)}-${row.area}`
      : `${row.time.slice(0, 10)}-${row.area}`
    groupedRows.set(key, row)
  })

  return [...groupedRows.values()].reverse()
})

const environmentStatusTone = (status: string) => status === '正常' ? 'ok' : status === '温湿度报警' ? 'danger' : 'warn'

const environmentAvgTemperature = computed(() => {
  const rows = realtimeEnvironmentRecords.value
  if (!rows.length) return '—'
  return `${(rows.reduce((total, item) => total + item.temperature, 0) / rows.length).toFixed(1)}℃`
})

const environmentAvgHumidity = computed(() => {
  const rows = realtimeEnvironmentRecords.value
  if (!rows.length) return '—'
  return `${Math.round(rows.reduce((total, item) => total + item.humidity, 0) / rows.length)}%`
})

const environmentTrendAvgTemperature = computed(() => {
  const rows = environmentChartSourceRecords.value
  if (!rows.length) return '—'
  return `${(rows.reduce((total, item) => total + item.temperature, 0) / rows.length).toFixed(1)}℃`
})

const environmentTrendAvgHumidity = computed(() => {
  const rows = environmentChartSourceRecords.value
  if (!rows.length) return '—'
  return `${Math.round(rows.reduce((total, item) => total + item.humidity, 0) / rows.length)}%`
})

const environmentWarningCount = computed(() => realtimeEnvironmentRecords.value.filter((item) => item.status !== '正常').length)

const environmentChartRows = computed<EnvironmentChartPoint[]>(() => {
  const activeRange = environmentRanges.find((item) => item.key === activeEnvironmentRange.value)
  const maxPoints = activeRange?.points ?? 8
  const map = new Map<string, EnvironmentRecord[]>()
  environmentChartSourceRecords.value.forEach((record) => {
    let key = ''
    if (activeEnvironmentRange.value === 'day') key = record.time.slice(11, 16)
    else if (activeEnvironmentRange.value === 'week' || activeEnvironmentRange.value === 'month') key = record.time.slice(0, 10)
    else key = record.time.slice(0, 7)
    map.set(key, [...(map.get(key) ?? []), record])
  })

  const groups = [...map.entries()].slice(-maxPoints)
  if (!groups.length) return []
  const tempValues = groups.map(([, rows]) => rows.reduce((total, item) => total + item.temperature, 0) / rows.length)
  const humidityValues = groups.map(([, rows]) => rows.reduce((total, item) => total + item.humidity, 0) / rows.length)
  const tempMin = Math.min(...tempValues, 20)
  const tempMax = Math.max(...tempValues, 32)
  const humidityMin = Math.min(...humidityValues, 40)
  const humidityMax = Math.max(...humidityValues, 70)

  return groups.map(([, rows], index) => {
    const avgTemp = Number((rows.reduce((total, item) => total + item.temperature, 0) / rows.length).toFixed(1))
    const avgHumidity = Math.round(rows.reduce((total, item) => total + item.humidity, 0) / rows.length)
    const tempPoint = chartPoint(index, avgTemp, tempMin, tempMax, groups.length)
    const humidityPoint = chartPoint(index, avgHumidity, humidityMin, humidityMax, groups.length)
    return {
      label: formatEnvironmentLabel(activeEnvironmentRange.value, rows[rows.length - 1].time),
      date: rows[rows.length - 1].time,
      temperature: avgTemp,
      humidity: avgHumidity,
      x: tempPoint.x,
      temperatureY: tempPoint.y,
      humidityY: humidityPoint.y,
    }
  })
})

const chartPoint = (index: number, value: number, min: number, max: number, total: number) => {
  const plotBottom = environmentChartLabelY - 30
  const x = total <= 1
    ? environmentChartWidth / 2
    : environmentChartPaddingX + (index / (total - 1)) * (environmentChartWidth - environmentChartPaddingX * 2)
  const y = environmentChartPaddingY + (1 - (value - min) / Math.max(1, max - min)) * (plotBottom - environmentChartPaddingY)
  return { x: Number(x.toFixed(1)), y: Number(y.toFixed(1)) }
}

const environmentTemperaturePoints = computed(() => environmentChartRows.value.map((item) => `${item.x},${item.temperatureY}`).join(' '))
const environmentHumidityPoints = computed(() => environmentChartRows.value.map((item) => `${item.x},${item.humidityY}`).join(' '))

const environmentPointPosition = (item: EnvironmentChartPoint, type: 'temperature' | 'humidity') => ({
  cx: item.x,
  cy: type === 'temperature' ? item.temperatureY : item.humidityY,
})

const showEnvironmentAxisLabel = (index: number, total: number) => {
  if (activeEnvironmentRange.value !== 'month') return true
  return index === 0 || index === total - 1 || index % 3 === 0
}

watch([activeEnvironmentRange, activeEnvironmentArea, activeEnvironmentAlarmType], () => {
  hoveredEnvironmentPoint.value = null
})

const reagentStockDefaults: ReagentStock[] = [
  { code: 'SJ-PT-001', name: '无水乙醇', type: '普通试剂', spec: '500ml/瓶', stock: 18, unit: '瓶', threshold: 8, location: '普通试剂柜A01', keeper: '质检员A', status: '库存正常' },
  { code: 'SJ-PT-002', name: '氢氧化钠标准溶液', type: '普通试剂', spec: '0.1mol/L', stock: 6, unit: '瓶', threshold: 8, location: '普通试剂柜A02', keeper: '王帅', status: '库存偏低' },
  { code: 'SJ-PT-003', name: '酚酞指示剂', type: '普通试剂', spec: '25g/瓶', stock: 10, unit: '瓶', threshold: 5, location: '普通试剂柜A03', keeper: '李审核', status: '库存正常' },
  { code: 'SJ-PT-004', name: '甲基红指示剂', type: '普通试剂', spec: '25g/瓶', stock: 8, unit: '瓶', threshold: 4, location: '普通试剂柜A01', keeper: '王帅', status: '库存正常' },
  { code: 'SJ-PT-005', name: '石油醚', type: '普通试剂', spec: '500ml/瓶', stock: 12, unit: '瓶', threshold: 6, location: '普通试剂柜A02', keeper: '质检员A', status: '库存正常' },
  { code: 'SJ-YZD-001', name: '盐酸', type: '易制毒试剂', spec: '500ml/瓶', stock: 4, unit: '瓶', threshold: 6, location: '易制毒双人双锁柜B01', keeper: '赵批准 / 质检员A', status: '库存偏低' },
  { code: 'SJ-YZD-002', name: '硫酸', type: '易制毒试剂', spec: '500ml/瓶', stock: 7, unit: '瓶', threshold: 4, location: '易制毒双人双锁柜B02', keeper: '赵批准 / 李审核', status: '重点管控' },
  { code: 'SJ-YZD-003', name: '高锰酸钾', type: '易制毒试剂', spec: '500g/瓶', stock: 3, unit: '瓶', threshold: 5, location: '易制毒双人双锁柜B03', keeper: '赵批准 / 王帅', status: '库存偏低' },
  { code: 'SJ-YZD-004', name: '丙酮', type: '易制毒试剂', spec: '500ml/瓶', stock: 9, unit: '瓶', threshold: 5, location: '易制毒双人双锁柜B01', keeper: '赵批准 / 李审核', status: '重点管控' },
  { code: 'SJ-YZD-005', name: '甲苯', type: '易制毒试剂', spec: '500ml/瓶', stock: 6, unit: '瓶', threshold: 4, location: '易制毒双人双锁柜B02', keeper: '赵批准 / 王帅', status: '重点管控' },
]

const reagentStocks = ref<ReagentStock[]>(reagentStockDefaults.map((item) => ({ ...item })))

const reagentCabinetSampledAt = currentDateTimeText()
const reagentCabinets = ref<ReagentCabinetEnvironment[]>([
  { code: 'A01', name: '普通试剂柜A01', type: '普通试剂柜', temperature: 24.8, humidity: 48, sampledAt: reagentCabinetSampledAt },
  { code: 'A02', name: '普通试剂柜A02', type: '普通试剂柜', temperature: 25.1, humidity: 50, sampledAt: reagentCabinetSampledAt },
  { code: 'A03', name: '普通试剂柜A03', type: '普通试剂柜', temperature: 27.6, humidity: 47, sampledAt: reagentCabinetSampledAt },
  { code: 'B01', name: '易制毒双人双锁柜B01', type: '易制毒双人双锁柜', temperature: 23.9, humidity: 46, sampledAt: reagentCabinetSampledAt },
  { code: 'B02', name: '易制毒双人双锁柜B02', type: '易制毒双人双锁柜', temperature: 24.2, humidity: 45, sampledAt: reagentCabinetSampledAt },
  { code: 'B03', name: '易制毒双人双锁柜B03', type: '易制毒双人双锁柜', temperature: 24.4, humidity: 49, sampledAt: reagentCabinetSampledAt },
])

const selectedReagentCabinet = computed(() => reagentCabinets.value.find((item) => item.code === selectedReagentCabinetCode.value) ?? reagentCabinets.value[0])
const reagentCabinetByCode = (code: string) => reagentCabinets.value.find((item) => item.code === code) ?? reagentCabinets.value[0]

const reagentCabinetStorageStandard = (cabinet: ReagentCabinetEnvironment) => cabinet.type === '易制毒双人双锁柜'
  ? { temperature: '15-25℃', minTemperature: 15, maxTemperature: 25, humidity: '30-60%', minHumidity: 30, maxHumidity: 60 }
  : { temperature: '15-25℃', minTemperature: 15, maxTemperature: 25, humidity: '30-60%', minHumidity: 30, maxHumidity: 60 }

const reagentCabinetAlert = (cabinet: ReagentCabinetEnvironment) => {
  const standard = reagentCabinetStorageStandard(cabinet)
  if (cabinet.temperature < standard.minTemperature || cabinet.temperature > standard.maxTemperature) return `温度${cabinet.temperature}℃超出标准 ${standard.temperature}`
  if (cabinet.humidity < standard.minHumidity || cabinet.humidity > standard.maxHumidity) return `湿度${cabinet.humidity}%超出标准 ${standard.humidity}`
  return ''
}

const reagentCabinetStatus = (cabinet: ReagentCabinetEnvironment, temperature: number, humidity: number) => {
  const standard = reagentCabinetStorageStandard(cabinet)
  const temperatureAlarm = temperature < standard.minTemperature || temperature > standard.maxTemperature
  const humidityAlarm = humidity < standard.minHumidity || humidity > standard.maxHumidity
  if (temperatureAlarm && humidityAlarm) return '温湿度报警'
  if (temperatureAlarm) return '温度报警'
  if (humidityAlarm) return '湿度报警'
  return '正常'
}

const reagentCabinetItems = (cabinet: ReagentCabinetEnvironment | undefined) => cabinet ? reagentStocks.value.filter((item) => item.location.includes(cabinet.code)) : []
const selectedReagentCabinetItems = computed(() => reagentCabinetItems(selectedReagentCabinet.value))
const filteredSelectedReagentCabinetItems = computed(() => {
  const keyword = reagentCabinetStockKeyword.value.trim().toLowerCase()
  return selectedReagentCabinetItems.value.filter((item) => {
    const matchesKeyword = !keyword || `${item.code} ${item.name} ${item.spec} ${item.keeper}`.toLowerCase().includes(keyword)
    const matchesStatus = reagentCabinetStockStatusFilter.value === 'all'
      || (reagentCabinetStockStatusFilter.value === 'normal' && !reagentCabinetAlert(selectedReagentCabinet.value))
      || (reagentCabinetStockStatusFilter.value === 'abnormal' && Boolean(reagentCabinetAlert(selectedReagentCabinet.value)))
    return matchesKeyword && matchesStatus
  })
})
const reagentCabinetAlertCount = computed(() => reagentCabinets.value.filter((cabinet) => reagentCabinetAlert(cabinet)).length)
const visibleReagentCabinets = computed(() => reagentCabinets.value.filter((cabinet) => cabinet.type === activeReagentCabinetType.value))
const visibleReagentCabinetRecords = computed(() => reagentCabinetRealtimeRecords.value.filter((record) => reagentCabinetByCode(record.cabinetCode).type === activeReagentCabinetType.value))

const buildReagentCabinetHistory = (): ReagentCabinetRecord[] => {
  const rows: ReagentCabinetRecord[] = []
  const sampleTimes = ['08:00', '12:00', '16:00', '20:00']
  const endDate = new Date(`${formatDateText(new Date())}T00:00:00`)

  for (let offset = 0; offset < 365; offset += 1) {
    const current = new Date(endDate)
    current.setDate(endDate.getDate() - offset)
    const dateText = localDateText(current)

    reagentCabinets.value.forEach((cabinet, cabinetIndex) => {
      sampleTimes.forEach((time, timeIndex) => {
        const seasonal = Math.sin((180 - offset) / 21) * 1.1
        const daily = [0, 0.5, 0.9, 0.3][timeIndex]
        let temperature = Number((23.1 + seasonal + daily + cabinetIndex * 0.18).toFixed(1))
        let humidity = Math.round(45 + Math.cos(offset / 12) * 3 + cabinetIndex * 0.7 + timeIndex)

        if (offset === 0 && cabinet.code === 'A03') {
          temperature = Number((26.7 + timeIndex * 0.3).toFixed(1))
        }
        if (offset === 5 && cabinet.code === 'B02' && timeIndex === 2) humidity = 63
        if (offset === 18 && cabinet.code === 'A01' && timeIndex === 3) temperature = 26.4

        rows.push({
          time: `${dateText} ${time}`,
          cabinetCode: cabinet.code,
          cabinetName: cabinet.name,
          temperature,
          humidity,
          status: reagentCabinetStatus(cabinet, temperature, humidity),
        })
      })
    })
  }

  return rows.sort((a, b) => a.time.localeCompare(b.time))
}

const reagentCabinetRecords = buildReagentCabinetHistory()
const reagentCabinetTrendRanges: { key: EnvironmentRange; label: string; points: number }[] = [
  { key: 'day', label: '当日', points: 8 },
  { key: 'week', label: '近一周', points: 7 },
  { key: 'month', label: '近一个月', points: 30 },
  { key: 'halfYear', label: '近一年', points: 12 },
]
const reagentCabinetRangeStartMap: Record<EnvironmentRange, string> = {
  day: formatDateText(new Date()),
  week: addDaysText(formatDateText(new Date()), -6),
  month: addDaysText(formatDateText(new Date()), -29),
  halfYear: addDaysText(formatDateText(new Date()), -364),
}
const activeReagentCabinetRangeLabel = computed(() => reagentCabinetTrendRanges.find((item) => item.key === activeReagentCabinetRange.value)?.label ?? '当日')
const reagentCabinetRealtimeRecords = computed<ReagentCabinetRecord[]>(() => reagentCabinets.value.map((cabinet) => ({
  time: cabinet.sampledAt,
  cabinetCode: cabinet.code,
  cabinetName: cabinet.name,
  temperature: cabinet.temperature,
  humidity: cabinet.humidity,
  status: reagentCabinetStatus(cabinet, cabinet.temperature, cabinet.humidity),
})))

const reagentCabinetRangeRecords = computed(() => reagentCabinetRecords
  .filter((item) => item.time.slice(0, 10) >= reagentCabinetRangeStartMap[activeReagentCabinetRange.value])
  .filter((item) => activeReagentCabinetFilter.value === '全部试剂柜' || item.cabinetCode === activeReagentCabinetFilter.value)
  .sort((a, b) => a.time.localeCompare(b.time)))

const reagentCabinetHistoryRows = computed(() => {
  const today = formatDateText(new Date())
  return reagentCabinetRecords
    .filter((item) => item.cabinetCode === selectedReagentCabinetCode.value && item.time.startsWith(today))
    .sort((a, b) => b.time.localeCompare(a.time))
})

const reagentCabinetChartRows = computed<EnvironmentChartPoint[]>(() => {
  const activeRange = reagentCabinetTrendRanges.find((item) => item.key === activeReagentCabinetRange.value)
  const maxPoints = activeRange?.points ?? 8
  const groupedRows = new Map<string, ReagentCabinetRecord[]>()
  reagentCabinetRangeRecords.value.forEach((record) => {
    let key = ''
    if (activeReagentCabinetRange.value === 'day') key = record.time.slice(11, 16)
    else if (activeReagentCabinetRange.value === 'week' || activeReagentCabinetRange.value === 'month') key = record.time.slice(0, 10)
    else key = record.time.slice(0, 7)
    groupedRows.set(key, [...(groupedRows.get(key) ?? []), record])
  })

  const groups = [...groupedRows.entries()].slice(-maxPoints)
  if (!groups.length) return []
  const temperatureValues = groups.map(([, rows]) => rows.reduce((total, item) => total + item.temperature, 0) / rows.length)
  const humidityValues = groups.map(([, rows]) => rows.reduce((total, item) => total + item.humidity, 0) / rows.length)
  const temperatureMin = Math.min(...temperatureValues, 20)
  const temperatureMax = Math.max(...temperatureValues, 32)
  const humidityMin = Math.min(...humidityValues, 40)
  const humidityMax = Math.max(...humidityValues, 70)

  return groups.map(([, rows], index) => {
    const temperature = Number((rows.reduce((total, item) => total + item.temperature, 0) / rows.length).toFixed(1))
    const humidity = Math.round(rows.reduce((total, item) => total + item.humidity, 0) / rows.length)
    const temperaturePoint = chartPoint(index, temperature, temperatureMin, temperatureMax, groups.length)
    const humidityPoint = chartPoint(index, humidity, humidityMin, humidityMax, groups.length)
    return {
      label: formatEnvironmentLabel(activeReagentCabinetRange.value, rows[rows.length - 1].time),
      date: rows[rows.length - 1].time,
      temperature,
      humidity,
      x: temperaturePoint.x,
      temperatureY: temperaturePoint.y,
      humidityY: humidityPoint.y,
    }
  })
})

const reagentCabinetTemperaturePoints = computed(() => reagentCabinetChartRows.value.map((item) => `${item.x},${item.temperatureY}`).join(' '))
const reagentCabinetHumidityPoints = computed(() => reagentCabinetChartRows.value.map((item) => `${item.x},${item.humidityY}`).join(' '))

const showReagentCabinetAxisLabel = (index: number, total: number) => {
  if (activeReagentCabinetRange.value === 'halfYear') return index === 0 || index === total - 1 || index % 2 === 0
  if (activeReagentCabinetRange.value !== 'month') return true
  return index === 0 || index === total - 1 || index % 3 === 0
}

const reagentCabinetTrendAvgTemperature = computed(() => {
  const rows = reagentCabinetRangeRecords.value
  if (!rows.length) return '—'
  return `${(rows.reduce((total, item) => total + item.temperature, 0) / rows.length).toFixed(1)}℃`
})

const reagentCabinetTrendAvgHumidity = computed(() => {
  const rows = reagentCabinetRangeRecords.value
  if (!rows.length) return '—'
  return `${Math.round(rows.reduce((total, item) => total + item.humidity, 0) / rows.length)}%`
})

watch([activeReagentCabinetRange, activeReagentCabinetFilter], () => {
  hoveredReagentCabinetPoint.value = null
})

const openReagentCabinetHistory = (cabinet: ReagentCabinetEnvironment) => {
  selectedReagentCabinetCode.value = cabinet.code
  hoveredReagentCabinetPoint.value = null
  showReagentCabinetHistory.value = true
}

const selectReagentCabinet = (cabinet: ReagentCabinetEnvironment) => {
  activeReagentCabinetType.value = cabinet.type
  selectedReagentCabinetCode.value = cabinet.code
  reagentCabinetStockKeyword.value = ''
  reagentCabinetStockStatusFilter.value = 'all'
  hoveredReagentCabinetPoint.value = null
}

const selectReagentCabinetType = (type: ReagentCabinetEnvironment['type']) => {
  activeReagentCabinetType.value = type
  const firstCabinet = reagentCabinets.value.find((cabinet) => cabinet.type === type)
  if (firstCabinet) selectReagentCabinet(firstCabinet)
}

const openReagentCabinetTrend = (cabinet: ReagentCabinetEnvironment) => {
  selectReagentCabinet(cabinet)
  activeReagentCabinetFilter.value = cabinet.code
  activeReagentCabinetRange.value = 'day'
  showReagentCabinetTrendDialog.value = true
}

const reagentEnvironmentStatus = (cabinet: ReagentCabinetEnvironment | undefined) => cabinet && reagentCabinetAlert(cabinet)
  ? '试剂环境异常'
  : '试剂环境正常'

const openReagentCabinetForm = () => {
  reagentCabinetForm.value = {
    code: '',
    name: '',
    type: activeReagentCabinetType.value,
    temperature: 22,
    humidity: 45,
  }
  showReagentCabinetForm.value = true
}

const addReagentCabinet = () => {
  const form = reagentCabinetForm.value
  const prefix = form.type === '普通试剂柜' ? 'A' : 'B'
  const typeCount = reagentCabinets.value.filter((cabinet) => cabinet.type === form.type).length
  let code = form.code.trim().toUpperCase() || `${prefix}${String(typeCount + 1).padStart(2, '0')}`
  while (reagentCabinets.value.some((cabinet) => cabinet.code === code)) code = `${prefix}${String(Number(code.slice(1)) + 1 || typeCount + 2).padStart(2, '0')}`
  const cabinet: ReagentCabinetEnvironment = {
    code,
    name: form.name.trim() || `${form.type}${code}`,
    type: form.type,
    temperature: Number.isFinite(Number(form.temperature)) ? Number(form.temperature) : 22,
    humidity: Number.isFinite(Number(form.humidity)) ? Number(form.humidity) : 45,
    sampledAt: currentDateTimeText(),
  }
  reagentCabinets.value.push(cabinet)
  reagentCabinetRecords.push({
    time: cabinet.sampledAt,
    cabinetCode: cabinet.code,
    cabinetName: cabinet.name,
    temperature: cabinet.temperature,
    humidity: cabinet.humidity,
    status: reagentCabinetStatus(cabinet, cabinet.temperature, cabinet.humidity),
  })
  selectReagentCabinet(cabinet)
  showReagentCabinetForm.value = false
}

const openReagentEntryForCabinet = () => {
  const cabinet = selectedReagentCabinet.value
  if (!cabinet) return
  reagentEntryForm.value = {
    action: '入库',
    reagentCode: '',
    name: '',
    spec: '',
    unit: '瓶',
    quantity: 0,
    threshold: 5,
    location: `${cabinet.name} ${cabinet.code}`,
    keeper: '',
    handler: '',
    time: '2026-07-12T09:00',
    type: cabinet.type === '易制毒双人双锁柜' ? '易制毒试剂' : '普通试剂',
    remark: '',
    supplier: '',
  }
  showReagentEntryForm.value = true
}

const reagentFlowRecords = ref<ReagentFlowRecord[]>([
  { recordNo: 'SJLC-2026070301', action: '入库', reagentCode: 'SJ-PT-001', reagentName: '无水乙醇', quantity: 10, unit: '瓶', location: '普通试剂柜A01', handler: '质检员A', time: '2026-07-03 08:30', remark: '采购到货验收', approvalStatus: '审批通过', approvalHistory: [{ title: '提交：留痕', user: '质检员A（编制人）', time: '2026-07-03 08:35' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-03 08:40' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-03 08:50' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-03 09:00' }] },
  { recordNo: 'SJLC-2026070302', action: '领用', reagentCode: 'SJ-YZD-001', reagentName: '盐酸', quantity: 1, unit: '瓶', location: '易制毒双人双锁柜B01', handler: '王帅', time: '2026-07-03 09:15', remark: '脂肪酸值检测', approvalStatus: '审批中', approvalStep: '审核人', processStatus: '审批中', stockApplied: false, approvalHistory: [{ title: '提交：领用申请', user: '王帅（编制人）', time: '2026-07-03 09:20' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-03 09:25' }] },
  { recordNo: 'SJLC-2026070401', action: '领用', reagentCode: 'SJ-YZD-002', reagentName: '硫酸', quantity: 1, unit: '瓶', location: '易制毒双人双锁柜B02', handler: '质检员A', time: '2026-07-04 09:10', remark: '理化检测领用', approvalStatus: '审批通过', processStatus: '已归还', returnRecordNo: 'SJLC-2026070501', stockApplied: true, approvalHistory: [{ title: '提交：领用申请', user: '质检员A（编制人）', time: '2026-07-04 09:12' }, { title: '审批完成：领用放行', user: '赵批准（批准人）', time: '2026-07-04 09:35' }] },
  { recordNo: 'SJLC-2026070402', action: '领用', reagentCode: 'SJ-YZD-003', reagentName: '高锰酸钾', quantity: 0.5, unit: '瓶', location: '易制毒双人双锁柜B03', handler: '李审核', time: '2026-07-04 14:20', remark: '氧化性检测领用', approvalStatus: '审批通过', processStatus: '领用中', stockApplied: true, approvalHistory: [{ title: '提交：领用申请', user: '李审核（编制人）', time: '2026-07-04 14:22' }, { title: '审批完成：领用放行', user: '赵批准（批准人）', time: '2026-07-04 14:45' }] },
  { recordNo: 'SJLC-2026070501', action: '归还', reagentCode: 'SJ-YZD-002', reagentName: '硫酸', quantity: 0.6, unit: '瓶', location: '易制毒双人双锁柜B02', handler: '质检员A', time: '2026-07-05 11:20', remark: '剩余试剂回柜', approvalStatus: '已完成', processStatus: '已归还', sourceRecordNo: 'SJLC-2026070401', inventoryStatus: '待盘点', inventoryRecordNo: 'SJLC-2026070501-PD', stockApplied: true },
  { recordNo: 'SJLC-2026070501-PD', action: '盘点', reagentCode: 'SJ-YZD-002', reagentName: '硫酸', quantity: 0.6, unit: '瓶', location: '易制毒双人双锁柜B02', handler: '系统', time: '2026-07-05 11:20', remark: '归还单 SJLC-2026070501 自动生成', approvalStatus: '待盘点', processStatus: '待盘点', sourceRecordNo: 'SJLC-2026070501', stockApplied: true },
  { recordNo: 'SJLC-2026070304', action: '盘点', reagentCode: 'SJ-YZD-003', reagentName: '高锰酸钾', quantity: 3, unit: '瓶', location: '易制毒双人双锁柜B03', handler: '李审核', time: '2026-07-03 16:30', remark: '触发采购提醒', approvalStatus: '已完成', processStatus: '已盘点', stockApplied: true },
])

const ensureReagentUsageDemoRecords = () => {
  const demoRecords: ReagentFlowRecord[] = [
    { recordNo: 'SJCG-2026071201', action: '采购', reagentCode: 'SJ-PT-004', reagentName: '甲基红指示剂', quantity: 8, unit: '瓶', location: '普通试剂柜A01', handler: '王帅', supplier: '镇江标准试剂有限公司', time: '2026-07-12 08:30', remark: '低库存补充采购', approvalStatus: '审批中', approvalStep: '审核人', processStatus: '审批中', stockApplied: false, approvalHistory: [{ title: '提交：采购申请', user: '王帅（编制人）', time: '2026-07-12 08:30' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-12 08:40' }] },
    { recordNo: 'SJCG-2026071101', action: '采购', reagentCode: 'SJ-PT-005', reagentName: '石油醚', quantity: 12, unit: '瓶', location: '普通试剂柜A02', handler: '质检员A', supplier: '江苏粮检试剂供应中心', time: '2026-07-11 09:00', remark: '月度采购计划', approvalStatus: '审批通过', stockApplied: false, approvalHistory: [{ title: '提交：采购申请', user: '质检员A（编制人）', time: '2026-07-11 09:00' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-11 11:00' }] },
    { recordNo: 'SJLC-2026070601', action: '领用', reagentCode: 'SJ-PT-001', reagentName: '无水乙醇', quantity: 2, unit: '瓶', location: '普通试剂柜A01', handler: '质检员A', time: '2026-07-06 09:20', remark: '水分检测前处理', approvalStatus: '审批通过', processStatus: '领用中', stockApplied: true, approvalHistory: [{ title: '提交：领用申请', user: '质检员A（编制人）', time: '2026-07-06 09:22' }, { title: '审批完成：领用放行', user: '赵批准（批准人）', time: '2026-07-06 09:40' }] },
    { recordNo: 'SJLC-2026070602', action: '领用', reagentCode: 'SJ-PT-002', reagentName: '氢氧化钠标准溶液', quantity: 1, unit: '瓶', location: '普通试剂柜A02', handler: '王帅', time: '2026-07-06 13:10', remark: '酸值测定领用', approvalStatus: '审批通过', processStatus: '领用中', stockApplied: true, approvalHistory: [{ title: '提交：领用申请', user: '王帅（编制人）', time: '2026-07-06 13:12' }, { title: '审批完成：领用放行', user: '赵批准（批准人）', time: '2026-07-06 13:30' }] },
    { recordNo: 'SJLC-2026070603', action: '领用', reagentCode: 'SJ-YZD-001', reagentName: '盐酸', quantity: 1, unit: '瓶', location: '易制毒双人双锁柜B01', handler: '李审核', time: '2026-07-06 15:00', remark: '脂肪酸值复测', approvalStatus: '审批通过', processStatus: '领用中', stockApplied: true, approvalHistory: [{ title: '提交：领用申请', user: '李审核（编制人）', time: '2026-07-06 15:02' }, { title: '审批完成：领用放行', user: '赵批准（批准人）', time: '2026-07-06 15:25' }] },
  ]
  const existing = new Set(reagentFlowRecords.value.map((item) => item.recordNo))
  const missing = demoRecords.filter((item) => !existing.has(item.recordNo))
  if (missing.length) reagentFlowRecords.value.push(...missing)
}

ensureReagentUsageDemoRecords()

const reagentStats = computed(() => [
  { label: '试剂总类', value: reagentStocks.value.length, tone: 'blue' },
  { label: '易制毒试剂', value: reagentStocks.value.filter((item) => item.type === '易制毒试剂').length, tone: 'red' },
  { label: '低库存提醒', value: reagentStocks.value.filter((item) => item.stock <= item.threshold).length, tone: 'orange' },
  { label: '待审批留痕', value: reagentFlowRecords.value.filter((item) => item.approvalStatus === '审批中' || item.approvalStatus === '未提交').length, tone: 'green' },
])

const reagentFlowSummary = computed(() => {
  const records = reagentFlowRecords.value
  return [
    { label: '采购', value: records.filter((item) => item.action === '采购').length, tone: 'purchase' },
    { label: '入库', value: records.filter((item) => item.action === '入库').length, tone: 'inbound' },
    { label: '领用', value: records.filter((item) => item.action === '领用').length, tone: 'usage' },
    { label: '归还', value: records.filter((item) => item.action === '归还').length, tone: 'return' },
    { label: '盘点', value: records.filter((item) => item.action === '盘点').length, tone: 'inventory' },
  ]
})

const openReagentFlowModule = (label: string) => {
  reagentTab.value = 'records'
  reagentProcessTab.value = label as ReagentActionType
}

const lowStockReagents = computed(() => reagentStocks.value.filter((item) => item.stock <= item.threshold))

const reagentInboundRecords = computed(() => reagentFlowRecords.value.filter((item) => item.action === '入库'))
const reagentUsageRecords = computed(() => reagentFlowRecords.value.filter((item) => item.action === '领用'))
const reagentReturnRecords = computed(() => reagentFlowRecords.value.filter((item) => item.action === '归还'))
const reagentActiveUsageRecords = computed(() => reagentUsageRecords.value.filter((item) => item.processStatus === '领用中'))
const reagentReturnWorkflowRecords = computed(() => reagentUsageRecords.value.filter((item) => item.processStatus === '领用中' || item.processStatus === '已归还'))
const reagentInventoryRecords = computed(() => reagentFlowRecords.value.filter((item) => item.action === '盘点'))

const resetReagentFlowForm = (action: ReagentActionType) => {
  reagentFlowForm.value = {
    action,
    reagentCode: '',
    name: '',
    spec: '',
    unit: '瓶',
    quantity: 0,
    threshold: 5,
    location: '',
    keeper: '',
    handler: currentLoginUser.value?.name ?? '',
    time: '2026-07-12T09:00',
    type: '普通试剂',
    remark: '',
    supplier: '',
  }
  reagentFlowSourceRecordNo.value = ''
}

const applyReagentStockToFlowForm = () => {
  const stock = reagentStocks.value.find((item) => item.code === reagentFlowForm.value.reagentCode)
  if (!stock) return
  reagentFlowForm.value.name = stock.name
  reagentFlowForm.value.spec = stock.spec
  reagentFlowForm.value.unit = stock.unit
  reagentFlowForm.value.threshold = stock.threshold
  reagentFlowForm.value.location = stock.location
  reagentFlowForm.value.keeper = stock.keeper
  reagentFlowForm.value.type = stock.type
}

const applyReagentFlowSource = () => {
  const source = reagentActiveUsageRecords.value.find((item) => item.recordNo === reagentFlowSourceRecordNo.value)
  if (!source) return
  const stock = reagentStocks.value.find((item) => item.code === source.reagentCode)
  reagentFlowForm.value.reagentCode = source.reagentCode
  reagentFlowForm.value.name = source.reagentName
  reagentFlowForm.value.unit = source.unit
  reagentFlowForm.value.location = source.location
  reagentFlowForm.value.quantity = source.quantity
  reagentFlowForm.value.spec = stock?.spec ?? ''
  reagentFlowForm.value.threshold = stock?.threshold ?? 5
  reagentFlowForm.value.keeper = stock?.keeper ?? ''
  reagentFlowForm.value.type = stock?.type ?? '普通试剂'
  reagentFlowForm.value.remark = `归还领用单 ${source.recordNo}`
}

const openReagentFlowForm = (action: ReagentActionType, sourceRecordNo = '') => {
  resetReagentFlowForm(action)
  if (action === '领用') {
    reagentFlowForm.value.reagentCode = reagentStocks.value[0]?.code ?? ''
    applyReagentStockToFlowForm()
  }
  if (action === '归还') {
    reagentFlowSourceRecordNo.value = sourceRecordNo || reagentActiveUsageRecords.value[0]?.recordNo || ''
    applyReagentFlowSource()
  }
  showReagentFlowForm.value = true
}

const openInboundFromProcurement = (record: ReagentFlowRecord) => {
  resetReagentFlowForm('入库')
  reagentFlowSourceRecordNo.value = record.recordNo
  reagentFlowForm.value = {
    ...reagentFlowForm.value,
    reagentCode: record.reagentCode,
    name: record.reagentName,
    spec: reagentStocks.value.find((item) => item.code === record.reagentCode)?.spec ?? '',
    unit: record.unit,
    quantity: record.quantity,
    location: record.location,
    remark: `采购单 ${record.recordNo} 到货入库`,
  }
  showReagentFlowForm.value = true
}

const reagentInboundRecordForProcurement = (record: ReagentFlowRecord) => reagentInboundRecords.value.find((item) => item.sourceRecordNo === record.recordNo)

const openReagentInboundReport = (record: ReagentFlowRecord) => {
  selectedReagentInboundRecord.value = record
  showReagentInboundReport.value = true
}

const updateReagentStockStatus = (stock: ReagentStock) => {
  stock.status = stock.stock <= stock.threshold ? '库存偏低' : stock.type === '易制毒试剂' ? '重点管控' : '库存正常'
}

const saveReagentFlow = () => {
  const form = reagentFlowForm.value
  const quantity = form.action === '归还'
    ? Math.max(0, Math.floor(Number(form.quantity) || 0))
    : Math.max(0, Number(form.quantity) || 0)
  const recordNo = `SJLC-${String(Date.now()).slice(-8)}`
  const time = formatDateTimeValue(form.time)
  const stock = reagentStocks.value.find((item) => item.code === form.reagentCode)

  if (form.action === '采购') {
    reagentFlowRecords.value.unshift({ recordNo, action: '采购', reagentCode: form.reagentCode || `CG-${String(Date.now()).slice(-6)}`, reagentName: form.name || '待采购试剂', quantity, unit: form.unit, location: form.location || '待入库', handler: form.handler || '系统', time, remark: form.remark || '新增采购申请', supplier: form.supplier || '待填写供应商', approvalStatus: '审批中', approvalStep: '编制人', processStatus: '审批中', stockApplied: false, approvalHistory: [{ title: '提交：采购申请', user: `${form.handler || currentLoginUser.value?.name || '编制人'}（编制人）`, time }] })
  }

  if (form.action === '入库') {
    if (stock) {
      stock.stock += quantity
      stock.spec = form.spec || stock.spec
      stock.location = form.location || stock.location
      stock.keeper = form.keeper || stock.keeper
      stock.threshold = form.threshold || stock.threshold
      updateReagentStockStatus(stock)
    } else {
      reagentStocks.value.push({
        code: form.reagentCode || `SJ-${form.type === '易制毒试剂' ? 'YZD' : 'PT'}-${String(reagentStocks.value.length + 1).padStart(3, '0')}`,
        name: form.name || '新试剂',
        type: form.type,
        spec: form.spec || '标准规格',
        stock: quantity,
        unit: form.unit,
        threshold: form.threshold || 5,
        location: form.location || '试剂柜',
        keeper: form.keeper || '质检员A',
        status: '库存正常',
      })
      updateReagentStockStatus(reagentStocks.value[reagentStocks.value.length - 1])
    }
    reagentFlowRecords.value.unshift({ recordNo, action: '入库', reagentCode: form.reagentCode || reagentStocks.value[reagentStocks.value.length - 1].code, reagentName: form.name || stock?.name || '新试剂', quantity, unit: form.unit, location: form.location || stock?.location || '试剂柜', handler: form.handler || '系统', time, remark: form.remark || '试剂入库登记', approvalStatus: '已入库', sourceRecordNo: reagentFlowSourceRecordNo.value || undefined, stockApplied: true })
  }

  if (form.action === '领用' && stock) {
    reagentFlowRecords.value.unshift({ recordNo, action: '领用', reagentCode: stock.code, reagentName: stock.name, quantity, unit: stock.unit, location: stock.location, handler: form.handler || '系统', time, remark: form.remark || '试剂领用申请', approvalStatus: '审批中', approvalStep: '编制人', processStatus: '审批中', stockApplied: false, approvalHistory: [{ title: '提交：领用申请', user: `${form.handler || currentLoginUser.value?.name || '编制人'}（编制人）`, time }] })
  }

  if (form.action === '归还') {
    const source = reagentActiveUsageRecords.value.find((item) => item.recordNo === reagentFlowSourceRecordNo.value)
    const sourceStock = source ? reagentStocks.value.find((item) => item.code === source.reagentCode) : undefined
    if (source && sourceStock) {
      const returnQuantity = Math.min(quantity, source.quantity)
      source.processStatus = '已归还'
      source.returnRecordNo = recordNo
      sourceStock.stock += returnQuantity
      updateReagentStockStatus(sourceStock)
      const inventoryRecordNo = `${recordNo}-PD`
      reagentFlowRecords.value.unshift(
        { recordNo: inventoryRecordNo, action: '盘点', reagentCode: source.reagentCode, reagentName: source.reagentName, quantity: returnQuantity, unit: source.unit, location: source.location, handler: '系统', time, remark: `归还单 ${recordNo} 自动生成`, approvalStatus: '待盘点', processStatus: '待盘点', sourceRecordNo: recordNo, stockApplied: true },
        { recordNo, action: '归还', reagentCode: source.reagentCode, reagentName: source.reagentName, quantity: returnQuantity, unit: source.unit, location: source.location, handler: form.handler || '系统', time, remark: form.remark || `归还领用单 ${source.recordNo}`, approvalStatus: '已完成', processStatus: '已归还', sourceRecordNo: source.recordNo, inventoryStatus: '待盘点', inventoryRecordNo, stockApplied: true },
      )
    }
  }

  resetReagentFlowForm(form.action)
  showReagentFlowForm.value = false
}

const openReagentInventoryConfirm = (record: ReagentFlowRecord, readOnly = false) => {
  selectedReagentInventoryRecord.value = record
  reagentInventoryConfirmReadOnly.value = readOnly
  reagentInventoryConfirmForm.value = {
    actualQuantity: record.inventoryActualQuantity ?? record.quantity,
    confirmer: currentLoginUser.value?.name ?? record.handler ?? '',
    remark: record.inventoryRemark ?? '',
  }
  showReagentInventoryConfirmDialog.value = true
}

const confirmReagentInventory = () => {
  const record = selectedReagentInventoryRecord.value
  if (!record) return
  const actualQuantity = Math.max(0, Number(reagentInventoryConfirmForm.value.actualQuantity) || 0)
  const result = actualQuantity === record.quantity ? '账实相符' : '存在差异'
  record.inventoryActualQuantity = actualQuantity
  record.inventoryResult = result
  record.inventoryConfirmedBy = reagentInventoryConfirmForm.value.confirmer || currentLoginUser.value?.name || '系统'
  record.inventoryConfirmedAt = currentDateTimeText()
  record.inventoryRemark = reagentInventoryConfirmForm.value.remark
  record.processStatus = '已盘点'
  record.approvalStatus = '已完成'
  const returnRecord = reagentReturnRecords.value.find((item) => item.recordNo === record.sourceRecordNo)
  if (returnRecord) returnRecord.inventoryStatus = '已盘点'
  showReagentInventoryConfirmDialog.value = false
  selectedReagentInventoryRecord.value = null
  reagentInventoryConfirmReadOnly.value = false
}

const reagentProcessStatusText = (record: ReagentFlowRecord) => {
  if (record.action === '采购') return record.approvalStatus === '审批通过' ? '采购审批通过' : record.approvalStatus
  if (record.action === '领用') return record.processStatus ?? '审批中'
  if (record.action === '归还') return record.processStatus ?? '已归还'
  if (record.action === '盘点') return record.processStatus ?? '已盘点'
  return '已入库'
}

const qualityResultValue = (record: SamplingRecord, item: string) => {
  if (record.reportResults?.[item]) return record.reportResults[item].judgement
  if (reportUnqualifiedItems(record).includes(item)) return '不合格'
  return '合格'
}

const qualityDetectValue = (record: SamplingRecord, item: string) => {
  if (record.reportResults?.[item]) return record.reportResults[item].detectValue
  if (!reportUnqualifiedItems(record).includes(item)) return ''
  const standard = activeQualityStandards(record).find((standardItem) => standardItem.item === item)?.standard
  return standard ? `${standard}，超标准值` : ''
}

const healthResultValue = (record: SamplingRecord, item: string) => {
  if (record.reportResults?.[item]) return record.reportResults[item].judgement
  if (reportUnqualifiedItems(record).includes(item)) return '不合格'
  return '合格'
}

const healthDetectValue = (record: SamplingRecord, item: string) => qualityDetectValue(record, item)

const reportResultValue = (item: string, field: keyof ReportResult) => {
  return selectedReportRecord.value.reportResults?.[item]?.[field] ?? ''
}

const parseStandardNumber = (value: string) => Number(value.match(/-?\d+(?:\.\d+)?/)?.[0] ?? Number.NaN)

const judgeByStandard = (standard: string, detectValue: string) => {
  const value = detectValue.trim()
  if (!value) return ''
  if (standard === '正常') return value === '正常' ? '合格' : '不合格'

  const standardNumber = parseStandardNumber(standard)
  const detectNumber = parseStandardNumber(value)
  if (Number.isNaN(standardNumber) || Number.isNaN(detectNumber)) return value === standard ? '合格' : '不合格'
  if (standard.includes('≤')) return detectNumber <= standardNumber ? '合格' : '不合格'
  if (standard.includes('≥')) return detectNumber >= standardNumber ? '合格' : '不合格'
  if (standard.includes('<')) return detectNumber < standardNumber ? '合格' : '不合格'
  if (standard.includes('>')) return detectNumber > standardNumber ? '合格' : '不合格'
  return detectNumber === standardNumber ? '合格' : '不合格'
}

const updateReportResult = (item: string, field: keyof ReportResult, value: string) => {
  const record = selectedReportRecord.value
  const standard = activeQualityStandards(record).find((standardItem) => standardItem.item === item)
  const detectValue = field === 'detectValue' ? value : record.reportResults?.[item]?.detectValue ?? ''
  record.reportResults = {
    ...(record.reportResults ?? {}),
    [item]: {
      detectValue,
      judgement: field === 'detectValue' && standard ? judgeByStandard(standard.standard, value) : record.reportResults?.[item]?.judgement ?? '',
      inspector: record.reportResults?.[item]?.inspector ?? '质检员A',
      standard: standard?.standard ?? record.reportResults?.[item]?.standard,
      type: standard?.type ?? record.reportResults?.[item]?.type,
      isKey: standard?.isKey ?? record.reportResults?.[item]?.isKey,
      [field]: value,
    },
  }
}

const instrumentIndicatorKeywords = (instrument: LabInstrument) => {
  if (instrument.name.includes('容重')) return ['容重']
  if (instrument.name.includes('水分') || instrument.name.includes('测水')) return ['水分']
  if (instrument.name.includes('杂质')) return ['杂质']
  if (instrument.name.includes('脂肪酸')) return ['脂肪酸值', '粗脂肪酸值', '酸价']
  if (instrument.name.includes('硬度')) return ['硬度']
  return []
}

const instrumentSupportedStandards = (instrument: LabInstrument) => {
  const keywords = instrumentIndicatorKeywords(instrument)
  return activeQualityStandards(selectedReportRecord.value).filter((standard) => keywords.some((keyword) => standard.item.includes(keyword)))
}

const instrumentAutoEntryRows = computed(() => labInstruments.value.map((instrument) => {
  const standards = instrumentSupportedStandards(instrument)
  return {
    instrument,
    standards,
    selectable: instrument.status !== '弃用' && standards.length > 0,
  }
}))

const selectedInstrumentItemCount = computed(() => new Set(instrumentAutoEntryRows.value
  .filter((row) => selectedInstrumentCodes.value.includes(row.instrument.code))
  .flatMap((row) => row.standards.map((standard) => standard.item))).size)

const buildInstrumentReading = (standard: QualityStandard, instrument: LabInstrument) => {
  if (standard.standard === '正常') return '正常'
  const standardValue = parseStandardNumber(standard.standard)
  if (Number.isNaN(standardValue)) return standard.standard
  const decimalCount = standard.standard.match(/\.(\d+)/)?.[1].length ?? 0
  const codeSeed = [...instrument.code].reduce((total, char) => total + char.charCodeAt(0), 0)
  const offset = (codeSeed % 7) / 100
  const value = standard.standard.includes('≥') || standard.standard.includes('>')
    ? standardValue * (1.06 + offset)
    : standardValue * (0.78 + offset)
  return value.toFixed(Math.min(Math.max(decimalCount, 1), 3))
}

const openInstrumentDataDialog = () => {
  selectedInstrumentCodes.value = []
  showInstrumentDataDialog.value = true
}

const applyInstrumentDetectionDataNow = () => {
  const target = selectedReportRecord.value
  const selectedRows = instrumentAutoEntryRows.value.filter((row) => selectedInstrumentCodes.value.includes(row.instrument.code) && row.selectable)
  if (!target || !selectedRows.length || !selectedInstrumentItemCount.value) return

  const groupedReadings = new Map<string, { standard: QualityStandard; readings: { value: string; instrument: LabInstrument }[] }>()
  selectedRows.forEach((row) => {
    row.standards.forEach((standard) => {
      const group = groupedReadings.get(standard.item) ?? { standard, readings: [] }
      group.readings.push({ value: buildInstrumentReading(standard, row.instrument), instrument: row.instrument })
      groupedReadings.set(standard.item, group)
    })
  })

  const capturedAt = currentDateTimeText()
  const nextResults = { ...(target.reportResults ?? buildDefaultReportResults(target)) }
  groupedReadings.forEach(({ standard, readings }, item) => {
    const numericValues = readings.map((reading) => Number(reading.value)).filter((value) => Number.isFinite(value))
    const detectValue = numericValues.length === readings.length
      ? (numericValues.reduce((total, value) => total + value, 0) / numericValues.length).toFixed(readings[0].value.split('.')[1]?.length ?? 0)
      : readings[0].value
    nextResults[item] = {
      ...(nextResults[item] ?? { inspector: currentLoginUser.value?.name ?? '质检员A' }),
      detectValue,
      judgement: judgeByStandard(standard.standard, detectValue),
      inspector: currentLoginUser.value?.name ?? '质检员A',
      standard: standard.standard,
      type: standard.type,
      isKey: standard.isKey,
      instrumentCode: readings.map((reading) => reading.instrument.code).join('、'),
      instrumentName: readings.map((reading) => reading.instrument.name).join('、'),
      capturedAt,
    }
  })
  target.reportResults = nextResults
  showInstrumentDataDialog.value = false
}

const applyInstrumentDetectionData = () => {
  const selectedRows = instrumentAutoEntryRows.value.filter((row) => selectedInstrumentCodes.value.includes(row.instrument.code) && row.selectable)
  if (!selectedRows.length || !selectedInstrumentItemCount.value) return
  confirmAction(
    `确认从 ${selectedRows.length} 台检化验设备获取 ${selectedInstrumentItemCount.value} 项检测数据并自动填充吗？`,
    applyInstrumentDetectionDataNow,
    { title: '设备数据获取确认', confirmText: '获取并填充' },
  )
}

const ledgerRecords = computed(() => {
  return reportRecords.value.filter((item) => {
    const receiveDate = String(item.receiveTime ?? '').slice(0, 10)
    const matchedGrain = !ledgerFilter.value.grainType || item.grainType === ledgerFilter.value.grainType
    const matchedSource = !ledgerFilter.value.source || item.source === ledgerFilter.value.source
    const matchedReason = !ledgerFilter.value.reason || item.reason === ledgerFilter.value.reason
    const matchedStart = !ledgerFilter.value.startDate || receiveDate >= ledgerFilter.value.startDate
    const matchedEnd = !ledgerFilter.value.endDate || receiveDate <= ledgerFilter.value.endDate
    return matchedGrain && matchedSource && matchedReason && matchedStart && matchedEnd
  })
})

const resetLedgerFilter = () => {
  ledgerFilter.value = { grainType: '', source: '', reason: '', startDate: '', endDate: '' }
}

const ledgerHandleMethod = (record: SamplingRecord) => {
  if (record.labelStatus === '留样') return '留样'
  if (record.labelStatus === '销样') return '销样'
  if (record.approvalStatus === '退回修改') return '退回修改'
  return '待处理'
}

const ledgerApproveUser = (record: SamplingRecord, role: string) => {
  return record.approvalHistory?.find((item) => item.user.includes(role))?.user.split('（')[0] ?? '—'
}

const latestHistoryTime = (record: SamplingRecord, keyword: string) => {
  return [...(record.approvalHistory ?? [])].reverse().find((item) => item.title.includes(keyword))?.time
}

const ledgerHandleTime = (record: SamplingRecord) => {
  if (record.labelStatus === '留样') return record.retainTime ?? record.handleTime ?? latestHistoryTime(record, '批准人：审批通过') ?? '—'
  if (record.labelStatus === '销样') return record.destroyTime ?? record.handleTime ?? latestHistoryTime(record, '转销样') ?? '—'
  if (record.approvalStatus === '退回修改') return record.handleTime ?? latestHistoryTime(record, '退回修改') ?? '—'
  return record.handleTime ?? record.approvalHistory?.[record.approvalHistory.length - 1]?.time ?? '—'
}

const qualityStandards: Record<string, Record<string, QualityStandard[]>> = {
  玉米: {
    入库验收: [
      { item: '镉 (mg/kg)', standard: '≤0.10', isKey: false, type: '卫生指标' },
      { item: '黄粒米 (%)', standard: '≤1.00', isKey: false, type: '质量等级' },
      { item: '水分 (g/100g)', standard: '≤14.00', isKey: true, type: '质量等级' },
      { item: '黄曲霉毒素B1 (μg/kg)', standard: '≤5.00', isKey: false, type: '质量等级' },
      { item: '色泽', standard: '正常', isKey: true, type: '质量等级' },
      { item: '气味', standard: '正常', isKey: true, type: '质量等级' },
      { item: '脂肪酸值 (mgKOH/100g)', standard: '≤50.00', isKey: false, type: '储存品质' },
      { item: '杂质 (%)', standard: '≤1.00', isKey: true, type: '质量等级' },
      { item: '铅 (mg/kg)', standard: '≤0.20', isKey: false, type: '卫生指标' },
      { item: '无机砷 (mg/kg)', standard: '≤0.20', isKey: false, type: '卫生指标' },
      { item: '三唑磷 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '毒死蜱 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '马拉硫磷 (mg/kg)', standard: '≤8', isKey: false, type: '卫生指标' },
      { item: '水胺硫磷 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '谷外糙米 (%)', standard: '≤2.0', isKey: false, type: '质量等级' },
      { item: '磷化物 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '品尝评分值 (分)', standard: '≥70.00', isKey: false, type: '储存品质' },
      { item: '汞 (mg/kg)', standard: '≤0.02', isKey: false, type: '卫生指标' },
    ],
    出库检测: [
      { item: '镉 (mg/kg)', standard: '≤0.10', isKey: false, type: '卫生指标' },
      { item: '黄粒米 (%)', standard: '≤1.00', isKey: false, type: '质量等级' },
      { item: '水分 (g/100g)', standard: '≤14.00', isKey: true, type: '质量等级' },
      { item: '黄曲霉毒素B1 (μg/kg)', standard: '≤5.00', isKey: false, type: '质量等级' },
      { item: '色泽', standard: '正常', isKey: true, type: '质量等级' },
      { item: '气味', standard: '正常', isKey: true, type: '质量等级' },
      { item: '脂肪酸值 (mgKOH/100g)', standard: '≤50.00', isKey: false, type: '储存品质' },
      { item: '杂质 (%)', standard: '≤1.00', isKey: true, type: '质量等级' },
      { item: '铅 (mg/kg)', standard: '≤0.20', isKey: false, type: '卫生指标' },
      { item: '无机砷 (mg/kg)', standard: '≤0.20', isKey: false, type: '卫生指标' },
      { item: '三唑磷 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '毒死蜱 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '马拉硫磷 (mg/kg)', standard: '≤8', isKey: false, type: '卫生指标' },
      { item: '水胺硫磷 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '谷外糙米 (%)', standard: '≤2.0', isKey: false, type: '质量等级' },
      { item: '磷化物 (mg/kg)', standard: '≤0.05', isKey: false, type: '卫生指标' },
      { item: '品尝评分值 (分)', standard: '≥70.00', isKey: false, type: '储存品质' },
      { item: '汞 (mg/kg)', standard: '≤0.02', isKey: false, type: '卫生指标' },
    ],
  },
  大豆: {
    入库验收: [
      { item: '水分 (g/100g)', standard: '≤13.50', isKey: true, type: '质量等级' },
      { item: '色泽', standard: '正常', isKey: true, type: '质量等级' },
      { item: '气味', standard: '正常', isKey: true, type: '质量等级' },
      { item: '破碎粒率 (%)', standard: '≤20.00', isKey: false, type: '质量等级，储存品质' },
      { item: '杂质 (%)', standard: '≤1.00', isKey: true, type: '质量等级' },
      { item: '粗脂肪酸值 (mg/g)', standard: '≤3.50', isKey: true, type: '储存品质' },
      { item: '霉变粒率 (%)', standard: '≤1.00', isKey: false, type: '质量等级，储存品质' },
      { item: '完整粒率 (%)', standard: '≥75.00', isKey: true, type: '质量等级' },
      { item: '损伤粒率 (%)', standard: '≤8.00', isKey: true, type: '质量等级' },
      { item: '热损伤粒率 (%)', standard: '≤1.00', isKey: false, type: '质量等级' },
      { item: '蛋白质含量 (g/100g)', standard: '≥33.00', isKey: true, type: '质量等级，储存品质' },
      { item: '赭曲霉毒素A (μg/kg)', standard: '≤5.00', isKey: false, type: '卫生指标' },
      { item: '镉 (mg/kg)', standard: '≤0.20', isKey: true, type: '卫生指标' },
    ],
    出库检测: [
      { item: '色泽', standard: '正常', isKey: true, type: '质量等级' },
      { item: '气味', standard: '正常', isKey: true, type: '质量等级' },
      { item: '水分 (g/100g)', standard: '≤13.50', isKey: false, type: '质量等级' },
      { item: '杂质 (%)', standard: '≤1.00', isKey: false, type: '质量等级' },
      { item: '完整粒率 (%)', standard: '≥75.00', isKey: false, type: '质量等级' },
      { item: '损伤粒率 (%)', standard: '≤8.00', isKey: false, type: '质量等级' },
      { item: '热损伤粒率 (%)', standard: '≤1.00', isKey: true, type: '质量等级' },
      { item: '破碎粒率 (%)', standard: '≤20.00', isKey: true, type: '质量等级，储存品质' },
      { item: '霉变粒率 (%)', standard: '≤1.00', isKey: false, type: '质量等级，储存品质' },
      { item: '粗脂肪酸值 (mg/g)', standard: '≤3.50', isKey: false, type: '储存品质' },
      { item: '蛋白质含量 (g/100g)', standard: '≥33.00', isKey: false, type: '质量等级，储存品质' },
      { item: '脂肪含量 (g/100g)', standard: '≥18.00', isKey: false, type: '质量等级' },
      { item: '赭曲霉毒素A (μg/kg)', standard: '≤5.00', isKey: false, type: '卫生指标' },
    ],
  },
}

const grainOptions = Object.keys(qualityStandards)
const inspectionReasons = ['入库验收', '出库检测', '春季普检', '秋季普检']
const internalReasons = inspectionReasons
const externalReasons = inspectionReasons
const isSupportedInspectionRecord = (record: Pick<SamplingRecord, 'reason'>) => inspectionReasons.includes(record.reason)
const warehouseOptions = ['49', '50']
const qualityStandardReason = (reason: string) => reason.includes('春季') || reason.includes('秋季') ? '出库检测' : reason
const activeQualityStandards = (record: Pick<SamplingRecord, 'grainType' | 'reason'>) => qualityStandards[record.grainType]?.[qualityStandardReason(record.reason)] ?? []
const qualityLevelItems = computed(() => activeQualityStandards(selectedReportRecord.value).filter((item) => item.type.includes('质量等级') && !item.type.includes('储存品质')))
const storageQualityItems = computed(() => activeQualityStandards(selectedReportRecord.value).filter((item) => item.type.includes('储存品质')))
const healthIndicatorItems = computed(() => activeQualityStandards(selectedReportRecord.value).filter((item) => item.type.includes('卫生指标')))
const inspectionKpis = computed(() => [
  { label: '扦样单', value: allSamplingRecords.value.length, tab: 'sampling', icon: ClipboardPenLine, tone: 'blue' },
  { label: '样品标签', value: labelRecords.value.length, tab: 'label', icon: Printer, tone: 'teal' },
  { label: '检验报告', value: completedReportRecords.value.length, tab: 'report', icon: ClipboardList, tone: 'violet' },
  { label: '样品台账', value: ledgerRecords.value.length, tab: 'ledger', icon: BookOpenText, tone: 'amber' },
  { label: '样品留样', value: retainRecords.value.length, tab: 'retain', icon: PackageCheck, tone: 'green' },
  { label: '销样', value: destroyRecords.value.length, tab: 'destroy', icon: Trash2, tone: 'red' },
])

const upcomingBusinessItems = computed(() => {
  const items = [
    { key: 'sampling', title: '扦样单审批确认待办', description: '送样或收样信息等待对应用户确认', records: samplingApprovalTodoRecords.value, icon: ClipboardPenLine, tone: 'blue', action: '审批确认' },
    { key: 'approval', title: '待审批报告审批', description: '检验报告等待当前用户审批', records: currentApprovalTodoRecords.value, icon: CheckCircle2, tone: 'orange', action: '审批报告' },
    { key: 'retain', title: '留样审批', description: '留样申请等待当前节点审批确认', records: retainRecords.value.filter((record) => canApproveRetain(record)), icon: PackageCheck, tone: 'green', action: '审批留样' },
    { key: 'destroy', title: '销样审批', description: '销样申请等待当前节点审批确认', records: destroyRecords.value.filter((record) => canApproveDestroy(record)), icon: Trash2, tone: 'red', action: '审批销样' },
  ]
  const total = items.reduce((sum, item) => sum + item.records.length, 0)
  return items.map((item) => ({ ...item, count: item.records.length, percent: total ? Math.round((item.records.length / total) * 100) : 0, record: item.records[0] }))
})

const upcomingBusinessTotal = computed(() => upcomingBusinessItems.value.reduce((sum, item) => sum + item.count, 0))
const upcomingBusinessChartSegments = computed(() => {
  let offset = 0
  return upcomingBusinessItems.value.filter((item) => item.count > 0).map((item) => {
    const segment = { ...item, offset }
    offset += item.percent
    return segment
  })
})

const inspectionTodoRows = computed(() => {
  const category = inspectionTodoDialog.value
  if (category === 'send') return allSamplingRecords.value.filter((record) => record.status === '扦样完成')
  if (category === 'receive') return allSamplingRecords.value.filter((record) => record.status === '已送样')
  if (category === 'inspect') return reportRecords.value.filter((record) => (record.labelStatus ?? '待检') === '待检')
  if (category === 'result') return reportRecords.value.filter((record) => record.labelStatus === '在检' && !record.approvalStep)
  if (category === 'approval') return reportRecords.value.filter((record) => record.labelStatus === '检毕' && Boolean(record.approvalStep))
  if (category === 'destroy') return destroyRecords.value
  return []
})

const inspectionTodoMeta = computed(() => {
  const category = inspectionTodoDialog.value
  const meta = {
    send: { title: '待送样业务', description: '扦样完成后，等待送交检化验室的样品' },
    receive: { title: '待收样业务', description: '已送样后，等待检化验室接收的样品' },
    inspect: { title: '待检业务', description: '已收样、等待扫码录入的样品' },
    result: { title: '待录入结果业务', description: '检测进行中、等待录入检验结果的样品' },
    approval: { title: '待审批业务', description: '质检结果已提交、等待审批流处理的样品' },
    destroy: { title: '待销样业务', description: '已确认销样并进入销样审批流程的样品' },
  }
  return category ? meta[category] : { title: '', description: '' }
})

const inspectionTodoItems = computed(() => {
  const records = allSamplingRecords.value
  const rows = [
    { key: 'send' as const, label: '待送样', count: records.filter((record) => record.status === '扦样完成').length, icon: Send, tone: 'blue' },
    { key: 'receive' as const, label: '待收样', count: records.filter((record) => record.status === '已送样').length, icon: PackageCheck, tone: 'teal' },
    { key: 'inspect' as const, label: '待检', count: reportRecords.value.filter((record) => (record.labelStatus ?? '待检') === '待检').length, icon: Microscope, tone: 'violet' },
    { key: 'result' as const, label: '待录入结果', count: reportRecords.value.filter((record) => record.labelStatus === '在检' && !record.approvalStep).length, icon: ClipboardPenLine, tone: 'amber' },
    { key: 'approval' as const, label: '待审批', count: reportRecords.value.filter((record) => record.labelStatus === '检毕' && Boolean(record.approvalStep)).length, icon: CheckCircle2, tone: 'orange' },
    { key: 'destroy' as const, label: '待销样', count: destroyRecords.value.length, icon: Trash2, tone: 'red' },
  ]
  const total = rows.reduce((sum, item) => sum + item.count, 0)
  const max = Math.max(...rows.map((item) => item.count), 1)
  return rows.map((item) => ({ ...item, percent: total ? Math.round((item.count / total) * 100) : 0, progress: Math.round((item.count / max) * 100) }))
})

const inspectionTodoTotal = computed(() => inspectionTodoItems.value.reduce((sum, item) => sum + item.count, 0))

const openInspectionTab = (tab: string) => {
  activeMenu.value = 'inspection'
  inspectionTab.value = tab
}

const openUpcomingBusiness = (item: { key: string; record?: SamplingRecord }) => {
  const record = item.record
  if (!record) return
  if (item.key === 'sampling') return openSamplingApprovalFlow(record)
  if (item.key === 'approval') return openApprovalFlow(record.sampleNo)
  if (item.key === 'retain') return openRetainApprovalFlow(record)
  if (item.key === 'destroy') return openDestroyApprovalFlow(record)
}

const openInspectionTodo = (category: InspectionTodoCategory) => {
  inspectionTodoDialog.value = category
}

const sampleStatusRows = computed(() => ['录入中', '扦样完成', '已送样', '待检', '在检', '检毕', '留样', '销样'].map((status) => {
  const count = allSamplingRecords.value.filter((item) => item.status === status || (item.status === '已收样' && (item.labelStatus ?? '待检') === status)).length
  const percent = allSamplingRecords.value.length ? Math.round((count / allSamplingRecords.value.length) * 100) : 0
  return { status, count, percent }
}).filter((item) => item.count > 0))

const buildReportNo = (record: SamplingRecord) => {
  const sourceCode = record.source === '内部扦样' ? 'NB' : 'WB'
  return `ZJBG-${sourceCode}-${record.sampleNo.replace(/[^0-9]/g, '')}`
}

const buildDefaultReportResults = (record: SamplingRecord) => {
  const unqualifiedItems = reportUnqualifiedMap[record.sampleNo] ?? []
  return activeQualityStandards(record).reduce<Record<string, ReportResult>>((result, standard) => {
    const isUnqualified = unqualifiedItems.includes(standard.item)
    result[standard.item] = {
      detectValue: isUnqualified ? `${standard.standard}，超标准值` : '',
      judgement: isUnqualified ? '不合格' : '',
      inspector: '质检员A',
      standard: standard.standard,
      type: standard.type,
      isKey: standard.isKey,
    }
    return result
  }, {})
}

const buildWarehouseMapReportResults = (grainType: string, reason: string, keyFailures: number, normalFailures: number) => {
  const record = { grainType, reason } as SamplingRecord
  const results = buildDefaultReportResults(record)
  const standards = activeQualityStandards(record)
  const failedStandards = [
    ...standards.filter((item) => item.isKey).slice(0, keyFailures),
    ...standards.filter((item) => !item.isKey).slice(0, normalFailures),
  ]

  failedStandards.forEach((standard) => {
    results[standard.item] = {
      ...results[standard.item],
      detectValue: `${standard.standard}，超出标准值`,
      judgement: '不合格',
    }
  })

  return results
}

const createCompleteSoybeanInboundReportResults = (): Record<string, ReportResult> => ({
  '水分 (g/100g)': { detectValue: '14.10', judgement: '不合格', inspector: '质检员A', standard: '≤13.50', type: '质量等级', isKey: true },
  色泽: { detectValue: '正常', judgement: '合格', inspector: '质检员A', standard: '正常', type: '质量等级', isKey: true },
  气味: { detectValue: '正常', judgement: '合格', inspector: '质检员A', standard: '正常', type: '质量等级', isKey: true },
  '破碎粒率 (%)': { detectValue: '12.30', judgement: '合格', inspector: '质检员A', standard: '≤20.00', type: '质量等级，储存品质', isKey: false },
  '杂质 (%)': { detectValue: '0.62', judgement: '合格', inspector: '质检员A', standard: '≤1.00', type: '质量等级', isKey: true },
  '粗脂肪酸值 (mg/g)': { detectValue: '2.74', judgement: '合格', inspector: '质检员A', standard: '≤3.50', type: '储存品质', isKey: true },
  '霉变粒率 (%)': { detectValue: '1.25', judgement: '不合格', inspector: '质检员A', standard: '≤1.00', type: '质量等级，储存品质', isKey: false },
  '完整粒率 (%)': { detectValue: '82.60', judgement: '合格', inspector: '质检员A', standard: '≥75.00', type: '质量等级', isKey: true },
  '损伤粒率 (%)': { detectValue: '5.10', judgement: '合格', inspector: '质检员A', standard: '≤8.00', type: '质量等级', isKey: true },
  '热损伤粒率 (%)': { detectValue: '0.30', judgement: '合格', inspector: '质检员A', standard: '≤1.00', type: '质量等级', isKey: false },
  '蛋白质含量 (g/100g)': { detectValue: '36.20', judgement: '合格', inspector: '质检员A', standard: '≥33.00', type: '质量等级，储存品质', isKey: true },
  '赭曲霉毒素A (μg/kg)': { detectValue: '1.60', judgement: '合格', inspector: '质检员A', standard: '≤5.00', type: '卫生指标', isKey: false },
  '镉 (mg/kg)': { detectValue: '0.08', judgement: '合格', inspector: '质检员A', standard: '≤0.20', type: '卫生指标', isKey: true },
})

const createCompleteCornSeasonReportResults = (): Record<string, ReportResult> => {
  const detectValues: Record<string, string> = {
    '镉 (mg/kg)': '0.04',
    '黄粒米 (%)': '0.36',
    '水分 (g/100g)': '13.20',
    '黄曲霉毒素B1 (μg/kg)': '2.10',
    色泽: '正常',
    气味: '正常',
    '脂肪酸值 (mgKOH/100g)': '32.60',
    '杂质 (%)': '0.48',
    '铅 (mg/kg)': '0.09',
    '无机砷 (mg/kg)': '0.05',
    '三唑磷 (mg/kg)': '0.01',
    '毒死蜱 (mg/kg)': '0.02',
    '马拉硫磷 (mg/kg)': '0.20',
    '水胺硫磷 (mg/kg)': '0.01',
    '谷外糙米 (%)': '1.10',
    '磷化物 (mg/kg)': '0.01',
    '品尝评分值 (分)': '82.00',
    '汞 (mg/kg)': '0.005',
  }

  return activeQualityStandards({ grainType: '玉米', reason: '秋季普检' }).reduce<Record<string, ReportResult>>((result, standard) => {
    result[standard.item] = {
      detectValue: detectValues[standard.item],
      judgement: '合格',
      inspector: '质检员A',
      standard: standard.standard,
      type: standard.type,
      isKey: standard.isKey,
    }
    return result
  }, {})
}

const inspectionTabs = [
  { key: 'overview', label: '流程总览' },
  { key: 'sampling', label: '扦样单管理' },
  { key: 'label', label: '样品标签管理' },
  { key: 'report', label: '检验报告管理' },
  { key: 'ledger', label: '样品台账管理' },
  { key: 'retain', label: '样品留样管理' },
  { key: 'destroy', label: '销样管理' },
]

const qualityWarningTestRecords: SamplingRecord[] = [
  { orderNo: 'QY20260705002', sampleNo: 'YP20260705002', sampleName: '50仓大豆二级预警测试样品', sampleCount: '2kg × 4', sampler: '李四', samplingDate: '2026-07-05', source: '内部扦样', status: '已收样', labelStatus: '检毕', sampleUnit: 'kg', approvalStatus: '审批中', approvalStep: '审核人', reason: '入库验收', grainType: '大豆', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '黑龙江', productionYear: '2026', storageDate: '2026-06-20', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', cargoPosition: '50仓1号货位', sender: '李四', sendTime: '2026-07-05 10:00', receiver: '质检员A', receiveTime: '2026-07-05 10:30', reportNo: 'ZJBG-NB-20260705002', reportMeta: { category: '监督检验', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '二级预警测试数据：关键不合格1项、普通不合格1项。' }, approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-05 11:10' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-05 11:30' }], reportResults: createCompleteSoybeanInboundReportResults() },
  { orderNo: 'QY20260706001', sampleNo: 'YP20260706001', sampleName: '49仓玉米春季普检样品', sampleCount: '2kg × 4', sampler: '张三', samplingDate: '2026-07-06', source: '内部扦样', status: '已收样', labelStatus: '检毕', sampleUnit: 'kg', approvalStatus: '审批通过', reason: '春季普检', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '江苏', productionYear: '2026', storageDate: '2026-06-18', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', cargoPosition: '49仓1号货位', surveyNo: 'CJP-20260706-49', sender: '张三', sendTime: '2026-07-06 09:00', receiver: '质检员A', receiveTime: '2026-07-06 09:30', reportNo: 'ZJBG-NB-20260706001', reportMeta: { category: '春季普检', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '春季普检测试数据，检测指标采用出库检测标准。' }, approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-06 10:10' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-06 11:00' }], reportResults: buildDefaultReportResults({ sampleNo: 'YP20260706001', grainType: '玉米', reason: '春季普检' } as SamplingRecord) },
  { orderNo: 'QY20260706002', sampleNo: 'YP20260706002', sampleName: '50仓大豆春季普检样品', sampleCount: '2kg × 4', sampler: '李四', samplingDate: '2026-07-06', source: '内部扦样', status: '已收样', labelStatus: '检毕', sampleUnit: 'kg', approvalStatus: '审批通过', reason: '春季普检', grainType: '大豆', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '黑龙江', productionYear: '2026', storageDate: '2026-06-20', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', cargoPosition: '50仓1号货位', surveyNo: 'CJP-20260706-50', sender: '李四', sendTime: '2026-07-06 10:00', receiver: '质检员A', receiveTime: '2026-07-06 10:30', reportNo: 'ZJBG-NB-20260706002', reportMeta: { category: '春季普检', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '春季普检测试数据：普通不合格1项，检测指标采用出库检测标准。' }, approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-06 11:10' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-06 12:00' }], reportResults: buildDefaultReportResults({ sampleNo: 'YP20260706002', grainType: '大豆', reason: '春季普检' } as SamplingRecord) },
  { orderNo: 'QY20260706003', sampleNo: 'YP20260706003', sampleName: '49仓玉米秋季普检样品', sampleCount: '2kg × 4', sampler: '王五', samplingDate: '2026-07-06', source: '内部扦样', status: '已收样', labelStatus: '留样', sampleUnit: 'kg', approvalStatus: '审批通过', reason: '秋季普检', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '江苏', productionYear: '2026', storageDate: '2026-06-22', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', cargoPosition: '49仓1号货位', surveyNo: 'QJP-20260706-49', sender: '王五', sendTime: '2026-07-06 13:00', receiver: '质检员A', receiveTime: '2026-07-06 13:30', reportNo: 'ZJBG-NB-20260706003', reportMeta: { category: '秋季普检', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '秋季普检测试数据，检测指标采用出库检测标准。' }, approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-06 14:10' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-06 15:00' }], retainTime: '2026-07-06 15:00', retainExpireDays: 30, reportResults: buildDefaultReportResults({ sampleNo: 'YP20260706003', grainType: '玉米', reason: '秋季普检' } as SamplingRecord) },
  { orderNo: 'QY20260706004', sampleNo: 'YP20260706004', sampleName: '50仓大豆秋季普检样品', sampleCount: '2kg × 4', sampler: '赵六', samplingDate: '2026-07-06', source: '内部扦样', status: '已收样', labelStatus: '检毕', sampleUnit: 'kg', approvalStatus: '审批中', approvalStep: '审核人', reason: '秋季普检', grainType: '大豆', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '黑龙江', productionYear: '2026', storageDate: '2026-06-25', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', cargoPosition: '50仓1号货位', surveyNo: 'QJP-20260706-50', sender: '赵六', sendTime: '2026-07-06 15:00', receiver: '质检员A', receiveTime: '2026-07-06 15:30', reportNo: 'ZJBG-NB-20260706004', reportMeta: { category: '秋季普检', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '秋季普检测试数据：关键不合格2项，检测指标采用出库检测标准。' }, approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-06 16:10' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-06 16:30' }], reportResults: buildDefaultReportResults({ sampleNo: 'YP20260706004', grainType: '大豆', reason: '秋季普检' } as SamplingRecord) },
  { orderNo: 'QY20260712001', sampleNo: 'YP20260712001', sampleName: '49仓玉米秋季普检完整测试样品', sampleCount: '2kg × 4', sampler: '张三', samplingDate: '2026-07-12', source: '内部扦样', status: '已收样', labelStatus: '检毕', sampleUnit: 'kg', approvalStatus: '审批通过', reason: '秋季普检', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '江苏', productionYear: '2026', storageDate: '2026-06-24', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', cargoPosition: '49仓1号货位', surveyNo: 'QJP-20260712-49', sender: '张三', sendTime: '2026-07-12 09:00', receiver: '质检员A', receiveTime: '2026-07-12 09:30', reportNo: 'ZJBG-NB-20260712001', reportMeta: { category: '秋季普检', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '完整测试数据：出库检测标准项均已录入，综合判定合格。' }, approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-12 10:10' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-12 10:30' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-12 10:50' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-12 11:10' }], reportResults: createCompleteCornSeasonReportResults() },
]

const createWorkflowStatusTestRecord = (overrides: Partial<SamplingRecord>): SamplingRecord => ({
  orderNo: '',
  sampleNo: '',
  sampleName: '质量流程测试样品',
  sampleCount: '2kg × 4',
  sampleUnit: 'kg',
  sampler: '张三',
  samplingDate: '2026-07-13',
  source: '内部扦样',
  status: '录入中',
  reason: '入库验收',
  grainType: '玉米',
  warehouseNo: '49',
  company: '中央储备粮镇江直属库有限公司',
  depot: '安鸿智慧粮库',
  representativeQuantity: '100.000',
  nature: '中央储备粮',
  origin: '江苏',
  productionYear: '2026',
  storageDate: '2026-06-26',
  packageType: '散装',
  retainCount: '1',
  inspectionCount: '1',
  keeper: '王保管',
  cargoPosition: '49仓1号货位',
  ...overrides,
})

const createDestroyTestRetainRecord = (index: number, overrides: Partial<SamplingRecord> = {}): SamplingRecord => createWorkflowStatusTestRecord({
  orderNo: `LY-TEST-${String(index).padStart(3, '0')}`,
  retainRecordId: `LY-TEST-${String(index).padStart(3, '0')}`,
  sampleNo: `YP-XY-TEST-${String(index).padStart(3, '0')}`,
  sampleName: `${49 + index}仓销样流程测试样品${index}`,
  status: '留样登记',
  labelStatus: '留样',
  approvalStatus: '审批通过',
  retainApprovalStatus: '留样完成',
  retainSampleStatus: '留样中',
  retainApprovalHistory: [
    { title: '留样送审', user: '王帅（编制人）', time: '2026-06-01 09:00' },
    { title: '确认留样：审批流程完成', user: '赵批准（批准人）', time: '2026-06-01 11:00' },
  ],
  samplingDate: '2026-05-31',
  receiveTime: '2026-05-31 16:00',
  retainTime: '2026-06-01 11:00',
  retainExpireAt: '2026-07-10',
  retainCount: '1',
  retainSpecification: '1kg密封留样袋',
  retainHandlingMethod: '留存',
  warehouseNo: String(49 + index),
  cargoPosition: `${49 + index}仓1号货位`,
  remark: '销样状态流转测试数据。',
  ...overrides,
})

const destroyTestRetainRecords: SamplingRecord[] = [
  createDestroyTestRetainRecord(1, { sampleName: '50仓过期待确认销样测试样品', retainExpireAt: '2026-07-10' }),
  createDestroyTestRetainRecord(2, { sampleName: '51仓销样审核人审批测试样品', retainSampleStatus: '销样中', retainExpireAt: '2026-07-08', retainHandlingMethod: '留样到期' }),
  createDestroyTestRetainRecord(3, { sampleName: '52仓销样负责人审批测试样品', retainSampleStatus: '销样中', retainExpireAt: '2026-07-05', retainHandlingMethod: '留样到期' }),
  createDestroyTestRetainRecord(4, { sampleName: '53仓销样最终确认测试样品', retainSampleStatus: '销样中', retainExpireAt: '2026-08-15', retainHandlingMethod: '主动销样' }),
  createDestroyTestRetainRecord(5, { sampleName: '54仓已销样测试样品', retainSampleStatus: '已销样', retainExpireAt: '2026-07-01', retainHandlingMethod: '留样到期', retainDisposalTime: '2026-07-12 15:30' }),
]

const createDestroyTestRecord = (source: SamplingRecord, step?: SamplingRecord['destroyApprovalStep'], completed = false): SamplingRecord => ({
  ...source,
  orderNo: `XX-TEST-${source.sampleNo.slice(-3)}`,
  status: '销样登记',
  labelStatus: '销样',
  reportNo: undefined,
  destroyRecordId: `XX-TEST-${source.sampleNo.slice(-3)}`,
  destroySourceRetainId: source.retainRecordId,
  destroyApprovalStatus: completed ? '已销样' : '审批中',
  destroyApprovalStep: completed ? undefined : step,
  destroyVariety: source.grainType,
  destroySource: source.retainHandlingMethod === '主动销样' ? '主动销样' : '留样到期',
  destroyTime: completed ? '2026-07-12 15:30' : undefined,
  destroyApprovalHistory: [
    { title: '销样送审', user: '王帅（编制人）', time: '2026-07-12 09:00' },
    ...(step === '负责人' || step === '批准人' || step === '确认销样' || completed ? [{ title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-12 10:00' }] : []),
    ...(step === '批准人' || step === '确认销样' || completed ? [{ title: '负责人：审批通过/同意', user: '周负责（负责人）', time: '2026-07-12 11:00' }] : []),
    ...(step === '确认销样' || completed ? [{ title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-12 14:00' }] : []),
    ...(completed ? [{ title: '确认销样：销样流程完成', user: '赵批准（批准人）', time: '2026-07-12 15:30' }] : []),
  ],
})

const destroyManagementTestRecords: SamplingRecord[] = [
  createDestroyTestRecord(destroyTestRetainRecords[1], '审核人'),
  createDestroyTestRecord(destroyTestRetainRecords[2], '负责人'),
  createDestroyTestRecord(destroyTestRetainRecords[3], '确认销样'),
  createDestroyTestRecord(destroyTestRetainRecords[4], undefined, true),
]

const workflowStatusTestRecords: SamplingRecord[] = [
  createWorkflowStatusTestRecord({ orderNo: 'QY20260713001', sampleNo: 'YP20260713001', sampleName: '49仓玉米录入中测试样品', status: '录入中', remark: '用于验证扦样单录入中状态。' }),
  createWorkflowStatusTestRecord({ orderNo: 'QY20260713002', sampleNo: 'YP20260713002', sampleName: '50仓大豆扦样完成测试样品', status: '扦样完成', grainType: '大豆', warehouseNo: '50', origin: '黑龙江', cargoPosition: '50仓1号货位', sampler: '李四', remark: '用于验证扦样完成待送样状态。' }),
  createWorkflowStatusTestRecord({ orderNo: 'QY20260713003', sampleNo: 'YP20260713003', sampleName: '49仓玉米已送样测试样品', status: '已送样', sender: '王五', sendTime: '2026-07-13 09:10', remark: '用于验证已送样待收样状态。' }),
  createWorkflowStatusTestRecord({ orderNo: 'QY20260713004', sampleNo: 'YP20260713004', sampleName: '50仓大豆留样测试样品', status: '已收样', labelStatus: '留样', approvalStatus: '审批通过', reason: '入库验收', grainType: '大豆', warehouseNo: '50', origin: '黑龙江', cargoPosition: '50仓1号货位', sampler: '李四', sender: '李四', sendTime: '2026-07-13 09:20', receiver: '质检员A', receiveTime: '2026-07-13 09:45', reportNo: 'ZJBG-NB-20260713004', reportMeta: { category: '监督检验', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '审批通过后留样，留样状态测试数据。' }, approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-13 10:10' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-13 10:25' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-13 10:40' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-13 11:00' }], retainTime: '2026-07-13 11:00', handleTime: '2026-07-13 11:00', retainExpireDays: 60, reportResults: buildDefaultReportResults({ grainType: '大豆', reason: '入库验收' } as SamplingRecord) }),
]

const createWarehouseMapSampleRecord = (warehouseNo: string, grainType: string, keyFailures: number, normalFailures: number): SamplingRecord => {
  const sampleNo = `MAP202607${warehouseNo}`
  return createWorkflowStatusTestRecord({
    orderNo: `QY20260714${warehouseNo}`,
    sampleNo,
    sampleName: `${warehouseNo}仓${grainType}当前检测样品`,
    samplingDate: '2026-07-14',
    status: '已收样',
    labelStatus: '检毕',
    approvalStatus: '审批通过',
    reason: '入库验收',
    grainType,
    warehouseNo,
    cargoPosition: `${warehouseNo}仓1号货位`,
    sender: '张三',
    sendTime: '2026-07-14 09:00',
    receiver: '质检员A',
    receiveTime: '2026-07-14 09:30',
    reportNo: `ZJBG-MAP-202607${warehouseNo}`,
    reportMeta: { category: '监督检验', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '仓房当前质量检测样品。' },
    warehouseMapSample: true,
    reportResults: buildWarehouseMapReportResults(grainType, '入库验收', keyFailures, normalFailures),
  })
}

const warehouseMapSampleRecords: SamplingRecord[] = [
  createWarehouseMapSampleRecord('49', '玉米', 0, 0),
  createWarehouseMapSampleRecord('50', '大豆', 0, 2),
  createWarehouseMapSampleRecord('51', '玉米', 0, 3),
  createWarehouseMapSampleRecord('52', '大豆', 2, 0),
  createWarehouseMapSampleRecord('53', '玉米', 0, 0),
]

const internalSamplingRecords = ref<SamplingRecord[]>([
  { orderNo: 'QY20260704002', sampleNo: 'YP20260704002', sampleName: '50仓大豆入库验收样品', sampleCount: '2kg × 4', sampler: '李四', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '待检', sampleUnit: 'kg', reason: '入库验收', grainType: '大豆', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '黑龙江', productionYear: '2026', storageDate: '2026-06-20', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', cargoPosition: '50仓1号货位', sender: '李四', sendTime: '2026-07-04 11:00', receiver: '质检员A', receiveTime: '2026-07-04 11:30' },
  ...qualityWarningTestRecords,
  ...workflowStatusTestRecords,
  ...warehouseMapSampleRecords,
].filter(isSupportedInspectionRecord))

const externalSamplingRecords = ref<SamplingRecord[]>([])

const normalizeReportLifecycle = (record: SamplingRecord) => {
  if (record.approvalStatus === '审批通过') {
    record.reportNo = record.reportNo ?? buildReportNo(record)
  } else {
    record.reportNo = undefined
  }
}

internalSamplingRecords.value.forEach(normalizeReportLifecycle)

const ensureQualityWarningTestRecords = () => {
  const existingSampleNos = new Set(internalSamplingRecords.value.map((record) => record.sampleNo))
  ;[...qualityWarningTestRecords, ...workflowStatusTestRecords, ...warehouseMapSampleRecords].filter(isSupportedInspectionRecord).forEach((record) => {
    if (!existingSampleNos.has(record.sampleNo)) {
      internalSamplingRecords.value.push(record)
    }
  })
}

const ensureDestroyManagementTestRecords = () => {
  const retainIds = new Set(internalRetainRecords.value.map((record) => record.retainRecordId))
  destroyTestRetainRecords.forEach((record) => {
    if (!retainIds.has(record.retainRecordId)) internalRetainRecords.value.push(record)
  })
  const destroyIds = new Set(internalDestroyRecords.value.map((record) => record.destroyRecordId))
  destroyManagementTestRecords.forEach((record) => {
    if (!destroyIds.has(record.destroyRecordId)) internalDestroyRecords.value.push(record)
  })
}

const loadPersistedState = () => {
  const ensureResponsibleAccount = () => {
    if (systemUsers.value.some((user) => user.username === 'fuzeren')) return
    systemUsers.value.unshift({ username: 'fuzeren', password: '123456', name: '\u5468\u8d1f\u8d23', role: '\u8d1f\u8d23\u4eba', department: '\u8d28\u91cf\u7ba1\u7406\u79d1', phone: '13800000005', status: '\u542f\u7528' })
  }
  const raw = localStorage.getItem(storageKey)
  if (!raw) {
    ensureResponsibleAccount()
    ensureDestroyManagementTestRecords()
    return
  }
  try {
    const state = JSON.parse(raw) as Partial<PersistedState>
    if (Array.isArray(state.internalSamplingRecords)) internalSamplingRecords.value = state.internalSamplingRecords.filter(isSupportedInspectionRecord)
    if (Array.isArray(state.externalSamplingRecords)) externalSamplingRecords.value = state.externalSamplingRecords.filter(isSupportedInspectionRecord)
    if (Array.isArray(state.retainRecords)) internalRetainRecords.value = state.retainRecords
    if (Array.isArray(state.destroyRecords)) internalDestroyRecords.value = state.destroyRecords
    ensureDestroyManagementTestRecords()
    internalDestroyRecords.value.forEach((record) => {
      if (record.destroyApprovalStatus === '销样完成') record.destroyApprovalStatus = '已销样'
      if (record.destroyApprovalStatus === '已销样') {
        const source = internalRetainRecords.value.find((retain) => retain.retainRecordId === record.destroySourceRetainId || retain.sampleNo === record.sampleNo)
        if (source) source.retainSampleStatus = '已销样'
      }
    })
    internalRetainRecords.value.forEach((record) => {
      const destroyRecord = internalDestroyRecords.value.find((destroy) => destroy.destroySourceRetainId === (record.retainRecordId ?? record.sampleNo) || destroy.sampleNo === record.sampleNo)
      if (destroyRecord?.destroyApprovalStatus === '已销样') record.retainSampleStatus = '已销样'
      else if (destroyRecord) record.retainSampleStatus = '销样中'
      else if (record.retainApprovalStatus === '留样完成') record.retainSampleStatus = '留样中'
      else record.retainSampleStatus = '审批中'
    })
    if (Array.isArray(state.systemUsers)) systemUsers.value = state.systemUsers
    ensureResponsibleAccount()
    if (Array.isArray(state.deletedLabelNos)) deletedLabelNos.value = state.deletedLabelNos
    if (Array.isArray(state.reagentStocks)) reagentStocks.value = state.reagentStocks
    reagentStockDefaults.forEach((defaultStock) => {
      if (!reagentStocks.value.some((stock) => stock.code === defaultStock.code)) reagentStocks.value.push({ ...defaultStock })
    })
    if (Array.isArray(state.reagentFlowRecords)) reagentFlowRecords.value = state.reagentFlowRecords
    if (Array.isArray(state.reagentCabinets) && state.reagentCabinets.length) reagentCabinets.value = state.reagentCabinets
    ensureReagentUsageDemoRecords()
    ensureQualityWarningTestRecords()
    allSamplingRecords.value.forEach((record) => {
      normalizeReportLifecycle(record)
      record.cargoPosition = normalizeCargoPositionValue(record)
      record.warehouseNo = cargoPositionWarehouseNo(record.cargoPosition)
    })
  } catch {
    localStorage.removeItem(storageKey)
    ensureResponsibleAccount()
    ensureQualityWarningTestRecords()
    ensureDestroyManagementTestRecords()
  }
}

const persistState = () => {
  const state: PersistedState = {
    internalSamplingRecords: internalSamplingRecords.value,
    externalSamplingRecords: externalSamplingRecords.value,
    retainRecords: internalRetainRecords.value,
    destroyRecords: internalDestroyRecords.value,
    systemUsers: systemUsers.value,
    deletedLabelNos: deletedLabelNos.value,
    reagentStocks: reagentStocks.value,
    reagentFlowRecords: reagentFlowRecords.value,
    reagentCabinets: reagentCabinets.value,
  }
  localStorage.setItem(storageKey, JSON.stringify(state))
}

onMounted(() => {
  loadPersistedState()
  loadWarehouseMapCalibration()
  persistState()
})

const handleStorageChange = (event: StorageEvent) => {
  if (event.key === storageKey) loadPersistedState()
}

onMounted(() => window.addEventListener('storage', handleStorageChange))
onUnmounted(() => window.removeEventListener('storage', handleStorageChange))

const warehousePositionTimer = window.setInterval(() => {
  activeWarehousePositionIndex.value = (activeWarehousePositionIndex.value + 1) % selectedWarehousePositions.value.length
}, 3000)

onUnmounted(() => window.clearInterval(warehousePositionTimer))

watch([internalSamplingRecords, externalSamplingRecords, internalRetainRecords, internalDestroyRecords, systemUsers, deletedLabelNos, reagentStocks, reagentFlowRecords, reagentCabinets], persistState, { deep: true })
watch(selectedWarehouse, () => {
  activeWarehousePositionIndex.value = 0
})

const openSamplingDetail = (orderNo: string) => {
  selectedSamplingNo.value = orderNo
  showSamplingDetail.value = true
}

const openSamplingLabel = (orderNo: string) => {
  selectedSamplingNo.value = orderNo
  showSamplingLabel.value = true
}

const openSampleLabelPrint = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  showSamplingLabel.value = true
}

const printSampleLabel = () => {
  const target = allSamplingRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value)
  printPage()
  if (target) target.labelPrintStatus = '已打印'
}

const confirmAction = (
  message: string,
  action: () => void,
  options: { title?: string; confirmText?: string; danger?: boolean } = {},
) => {
  confirmDialog.value = {
    title: options.title ?? '操作确认',
    message,
    confirmText: options.confirmText ?? '确认',
    danger: options.danger ?? false,
  }
  pendingConfirmAction.value = action
  showConfirmDialog.value = true
}

const closeConfirmDialog = () => {
  showConfirmDialog.value = false
  pendingConfirmAction.value = null
}

const acceptConfirmDialog = () => {
  const action = pendingConfirmAction.value
  showConfirmDialog.value = false
  pendingConfirmAction.value = null
  action?.()
}

const openInspectionScanDialog = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  inspectionScanValue.value = ''
  showInspectionScanDialog.value = true
}

const readInspectionSampleCode = () => {
  inspectionScanValue.value = displaySampleNo(selectedReportRecord.value)
}

const confirmInspectionScan = () => {
  const target = reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value)
  if (!target) return
  if (inspectionScanValue.value.trim() !== displaySampleNo(target)) return
  confirmAction(
    `确认扫码录入样品 ${displaySampleNo(target)} 并进入检验流程吗？`,
    () => {
      target.labelStatus = '在检'
      target.approvalStatus = '未提交'
      target.reportResults = target.reportResults ?? buildDefaultReportResults(target)
      target.inspectionScanTime = currentDateTimeText()
      target.scanOperator = currentLoginUser.value?.name ?? '系统'
      showInspectionScanDialog.value = false
    },
    { title: '扫码录入确认', confirmText: '确认录入' },
  )
}

const openReportForm = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  if (canEditReportResult(selectedReportRecord.value)) {
    const target = selectedReportRecord.value
    confirmAction(
      `确认进入样品 ${displaySampleNo(target)} 的质检结果录入吗？`,
      () => {
        reportMetaForm.value = {
          category: '监督检验',
          compiler: currentLoginUser.value?.name ?? '王帅',
          reviewer: '李审核',
          approver: '赵批准',
          remark: '',
          qualityGrade: qualityGradeText(target),
          qualified: qualifiedText(target),
          ...target.reportMeta,
        }
        showReportForm.value = true
      },
      { title: '结果录入确认', confirmText: '进入录入' },
    )
  }
}

const reportApprovalStatusText = (record: SamplingRecord) => {
  if (record.approvalStatus === '审批通过' && record.reportNo) return '审批通过 / 报告已生成'
  if (record.approvalStatus === '销样') return '审批未通过'
  if (record.approvalStatus === '退回修改') return '退回修改'
  return record.approvalStep ? '审批中' : record.approvalStatus ?? '未提交'
}

const reportProcessStatusText = (record: SamplingRecord) => {
  if (record.approvalStatus === '审批通过' && record.reportNo) return '质检报告已生成'
  if (record.approvalStatus === '销样' || record.labelStatus === '销样') return '审批未通过'
  if (record.approvalStatus === '退回修改') return '待修改结果'
  if (record.approvalStep) return `${record.approvalStep}审批中`
  if (record.labelStatus === '检毕') return '待提交审批'
  if (record.labelStatus === '在检') return '待录入结果'
  return '待扫码录入'
}

const reportProcessStatusTone = (record: SamplingRecord) => {
  if (record.approvalStatus === '审批通过' && record.reportNo) return 'complete'
  if (record.approvalStatus === '销样' || record.labelStatus === '销样') return 'failed'
  if (record.approvalStatus === '退回修改') return 'returned'
  if (record.approvalStep) return 'approval'
  if (record.labelStatus === '在检') return 'active'
  return 'pending'
}

const reportHasFormalReport = (record: SamplingRecord) => record.approvalStatus === '审批通过' && Boolean(record.reportNo)

const canEditReportResult = (record: SamplingRecord) => {
  return record.labelStatus === '在检' && !record.approvalStep
}

const addApprovalHistory = (record: SamplingRecord, title: string, opinion = '') => {
  const user = currentLoginUser.value
  record.approvalHistory = [
    ...(record.approvalHistory ?? []),
    { title, user: user ? `${user.name}（${user.role}）` : '系统', time: currentDateTimeText(), ...(opinion.trim() ? { opinion: opinion.trim() } : {}) },
  ]
}

const reportApprovalHistory = (record: SamplingRecord) => {
  return (record.approvalHistory ?? []).filter((item) => !item.title.startsWith('编制人：'))
}

const approvalFlowItems = (record: SamplingRecord) => {
  const history = reportApprovalHistory(record)
  return [
    { name: '结果送审', state: history.some((item) => item.title.includes('提交')) || record.approvalStep || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: Send, desc: '质检结果提交' },
    { name: '审核人审批', state: record.approvalStep === '审核人' ? 'active' : history.some((item) => item.title.includes('审核人')) || ['负责人', '批准人'].includes(String(record.approvalStep)) || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: ClipboardList, desc: '质检结果审核' },
    { name: '负责人审批', state: record.approvalStep === '负责人' ? 'active' : history.some((item) => item.title.includes('负责人')) || record.approvalStep === '批准人' || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: Edit3, desc: '质量负责人复核' },
    { name: '批准人审批', state: record.approvalStep === '批准人' ? 'active' : record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: CheckCircle2, desc: '批准放行' },
    { name: '质检报告生成', state: record.approvalStatus === '审批通过' && record.reportNo ? 'done' : 'todo', icon: BookOpenText, desc: record.reportNo ? '正式报告已生成' : '批准通过后生成' },
  ]
}

const openApprovalFlow = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  approvalOpinion.value = ''
  showApprovalFlow.value = true
}

const addDaysToDateText = (dateText: string, days: number) => {
  const date = new Date(`${dateText}T00:00:00`)
  date.setDate(date.getDate() + days)
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${date.getFullYear()}-${month}-${day}`
}

const resetInstrumentCategoryForm = () => {
  instrumentCategoryForm.value = {
    type: '',
    description: '',
    storageArea: '',
    note: '',
  }
}

const resetInstrumentEntryForm = () => {
  instrumentEntryForm.value = {
    type: activeInstrumentType.value,
    name: '',
    model: '',
    location: '',
    custodian: '',
    calibration: '',
  }
}

const addInstrumentCategory = () => {
  const form = instrumentCategoryForm.value
  const type = normalizeLabInstrumentType(form.type)
  if (!type) return
  const existing = labInstrumentCategories.value.find((item) => item.type === type)
  const payload = {
    type,
    description: form.description || '未填写类别说明',
    storageArea: form.storageArea || '未指定库区',
    note: form.note || '未填写备注',
  }

  if (existing) {
    Object.assign(existing, payload)
  } else {
    labInstrumentCategories.value.push(payload)
  }

  activeInstrumentType.value = type
  resetInstrumentCategoryForm()
  showInstrumentCategoryForm.value = false
}

const addInstrument = () => {
  const form = instrumentEntryForm.value
  const type = normalizeLabInstrumentType(form.type) || activeInstrumentType.value
  if (!labInstrumentCategories.value.some((item) => item.type === type)) {
    labInstrumentCategories.value.push({
      type,
      description: '新增设备类别',
      storageArea: form.location || '未指定库区',
      note: '自动补充',
    })
  }

  const idx = labInstruments.value.length + 1
  const calibrationDate = form.calibration || '2026-07-12'
  labInstruments.value.push({
    code: `YQ-XZ-${String(idx).padStart(3, '0')}`,
    name: form.name || `新仪器${idx}`,
    model: form.model || '标准型号',
    location: form.location || '检化验室',
    custodian: form.custodian || '质检员A',
    status: '在用',
    calibration: calibrationDate,
    maintenance: addDaysToDateText(calibrationDate, 30),
    lastMaintenance: calibrationDate,
    maintenanceType: '月度维护',
    maintenanceHandler: '',
    usage: 50,
    type,
    lastCalibration: calibrationDate,
  })
  activeInstrumentType.value = type
  resetInstrumentEntryForm()
  showInstrumentEntryForm.value = false
}

const addReagentLedger = () => {
  const form = reagentEntryForm.value
  const quantity = Math.max(0, Number(form.quantity) || 0)
  const idx = reagentStocks.value.length + 1
  const prefix = form.type === '易制毒试剂' ? 'YZD' : 'PT'
  const reagentCode = form.reagentCode || `SJ-${prefix}-${String(idx).padStart(3, '0')}`
  const reagentName = form.name || `新试剂${idx}`
  const target = reagentStocks.value.find((item) => item.code === reagentCode || item.name === reagentName)

  if (target) {
    target.stock = quantity || target.stock
    target.spec = form.spec || target.spec
    target.location = form.location || target.location
    target.keeper = form.keeper || target.keeper
    target.threshold = form.threshold || target.threshold
    target.status = target.stock <= target.threshold ? '库存偏低' : target.type === '易制毒试剂' ? '重点管控' : '库存正常'
  } else if (form.action === '入库') {
    reagentStocks.value.push({
      code: reagentCode,
      name: reagentName,
      type: form.type,
      spec: form.spec || '标准规格',
      stock: quantity || 10,
      unit: form.unit,
      threshold: form.threshold || 5,
      location: form.location || '试剂柜',
      keeper: form.keeper || '质检员A',
      status: (quantity || 10) <= (form.threshold || 5) ? '库存偏低' : form.type === '易制毒试剂' ? '重点管控' : '库存正常',
    })
  }

  reagentEntryForm.value = {
    action: '入库',
    reagentCode: '',
    name: '',
    spec: '',
    unit: '瓶',
    quantity: 0,
    threshold: 5,
    location: '',
    keeper: '',
    handler: '',
    time: '2026-07-12T09:00',
    type: '普通试剂',
    remark: '',
    supplier: '',
  }
  showReagentEntryForm.value = false
}

const saveReportResult = () => {
  const target = reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value)
  if (!target || !canEditReportResult(target)) return
  confirmAction(
    `确认保存样品 ${displaySampleNo(target)} 的质检结果吗？`,
    () => {
      target.labelStatus = '在检'
      target.approvalStatus = '未提交'
      target.reportResults = target.reportResults ?? buildDefaultReportResults(target)
      target.reportMeta = { ...reportMetaForm.value }
      showReportForm.value = false
    },
    { title: '保存结果确认', confirmText: '确认保存' },
  )
}

const submitReportResult = () => {
  const target = reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value)
  if (!target || !canEditReportResult(target)) return
  confirmAction(
    `确认提交样品 ${displaySampleNo(target)} 的质检结果进入审批吗？提交后将暂时不能修改。`,
    () => {
      target.labelStatus = '检毕'
      target.approvalStatus = '审批中'
      target.approvalStep = '审核人'
      target.reportNo = undefined
      target.reportResults = target.reportResults ?? buildDefaultReportResults(target)
      target.reportMeta = { ...reportMetaForm.value }
      target.approvalHistory = [
        ...(target.approvalHistory ?? []).filter((item) => !item.title.includes('提交：送审')),
        { title: '提交：送审', user: `${currentLoginUser.value?.name ?? '编制人'}（编制人）`, time: currentDateTimeText() },
      ]
      showReportForm.value = false
    },
    { title: '提交审批确认', confirmText: '确认提交' },
  )
}

const openReportView = (sampleNo: string) => {
  const target = reportRecords.value.find((item) => item.sampleNo === sampleNo)
  if (!target || !reportHasFormalReport(target)) return
  selectedSamplingNo.value = sampleNo
  reportViewMode.value = 'report'
  showReportView.value = true
}

const openQualityResultView = (sampleNo: string) => {
  const target = reportRecords.value.find((item) => item.sampleNo === sampleNo)
  if (!target?.reportResults) return
  selectedSamplingNo.value = sampleNo
  reportViewMode.value = 'result'
  showApprovalFlow.value = false
  showReportView.value = true
}

const canApproveReport = (record: SamplingRecord) => {
  return record.labelStatus === '检毕' && record.approvalStep === currentLoginUser.value?.role
}

const currentApprovalTodoRecords = computed(() => reportRecords.value.filter((record) => canApproveReport(record)))

const inspectionOverviewDrilldownMeta = computed(() => {
  const mode = inspectionOverviewDrilldown.value
  if (mode === 'sampling') return { title: '今日扦样单', description: '查看指定日期的扦样单表单信息' }
  if (mode === 'pending') return { title: '待检样品报告', description: '查看当前待检状态的检验报告表单信息' }
  if (mode === 'approval') return { title: '待审报告', description: '仅显示当前登录用户可审批的检验报告' }
  if (mode === 'retain') return { title: '样品留样信息', description: '查看留样数量、留样时间和到期状态' }
  return { title: '销样管理信息', description: '查看已转入销样管理的样品信息' }
})

const inspectionOverviewRows = computed(() => {
  const mode = inspectionOverviewDrilldown.value
  let rows: SamplingRecord[] = []
  if (mode === 'sampling') rows = allSamplingRecords.value.filter((item) => item.samplingDate === inspectionOverviewTodayDate.value)
  if (mode === 'pending') rows = reportRecords.value.filter((item) => (item.labelStatus ?? '待检') === '待检')
  if (mode === 'approval') rows = currentApprovalTodoRecords.value
  if (mode === 'retain') rows = retainRecords.value
  if (mode === 'destroy') rows = destroyRecords.value

  const filters = inspectionOverviewFilters.value
  return rows.filter((item) => {
    const keyword = filters.keyword.trim().toLowerCase()
    const keywordMatches = !keyword || [item.sampleNo, item.orderNo, item.reportNo, item.sampleName].some((value) => String(value ?? '').toLowerCase().includes(keyword))
    const warehouseMatches = !filters.warehouseNo || item.warehouseNo === filters.warehouseNo
    const sourceMatches = !filters.source || item.source === filters.source
    const reasonMatches = !filters.reason || item.reason === filters.reason
    const retainMatches = !filters.retainStatus || (filters.retainStatus === '已到期' ? retainStatusLevel(item) === 'expired' : retainStatusLevel(item) !== 'expired')
    return keywordMatches && warehouseMatches && sourceMatches && reasonMatches && retainMatches
  })
})

const inspectionOverviewWarehouseOptions = computed(() => [...new Set(allSamplingRecords.value.map((item) => item.warehouseNo).filter(Boolean))])
const inspectionOverviewReasonOptions = computed(() => [...new Set(allSamplingRecords.value.map((item) => item.reason).filter(Boolean))])

const openInspectionOverviewDrilldown = (mode: InspectionOverviewDrilldown) => {
  inspectionOverviewDrilldown.value = mode
  inspectionOverviewFilters.value = { keyword: '', warehouseNo: '', source: '', reason: '', retainStatus: '' }
  if (mode === 'sampling') inspectionOverviewTodayDate.value = '2026-07-04'
}

const executeReportApproval = (target: SamplingRecord, action: 'pass' | 'return', opinion = '') => {
  if (action === 'return') {
    addApprovalHistory(target, `${target.approvalStep}：不同意，退回修改`, opinion)
    target.labelStatus = '在检'
    target.approvalStatus = '退回修改'
    target.approvalStep = undefined
    target.reportNo = undefined
    target.handleTime = target.approvalHistory?.[target.approvalHistory.length - 1]?.time
    return
  }

  if (target.approvalStep === '审核人') {
    addApprovalHistory(target, '审核人：审批通过/同意', opinion)
    target.approvalStep = '负责人'
    target.approvalStatus = '审批中'
  } else if (target.approvalStep === '负责人') {
    addApprovalHistory(target, '负责人：审批通过/同意', opinion)
    target.approvalStep = '批准人'
    target.approvalStatus = '审批中'
  } else if (target.approvalStep === '批准人') {
    addApprovalHistory(target, '批准人：审批通过/同意', opinion)
    target.labelStatus = '检毕'
    target.approvalStatus = '审批通过'
    target.approvalStep = undefined
    target.reportNo = buildReportNo(target)
    addApprovalHistory(target, '质检报告生成：正式报告已生成')
    target.handleTime = target.approvalHistory?.[target.approvalHistory.length - 1]?.time
  }
}

const approveReport = (sampleNo: string, action: 'pass' | 'return') => {
  selectedSamplingNo.value = sampleNo
  const target = reportRecords.value.find((item) => item.sampleNo === sampleNo)
  if (!target || !canApproveReport(target)) return

  const opinion = approvalOpinion.value.trim()
  const confirmMessage = action === 'pass'
    ? `确认以“${target.approvalStep}”身份通过样品 ${displaySampleNo(target)} 的质检结果吗？`
    : `确认退回样品 ${displaySampleNo(target)} 的质检结果进行修改吗？`
  confirmAction(
    confirmMessage,
    () => {
      executeReportApproval(target, action, opinion)
      approvalOpinion.value = ''
    },
    {
      title: action === 'pass' ? '审批通过确认' : '退回修改确认',
      confirmText: action === 'pass' ? '确认通过' : '确认退回',
      danger: action === 'return',
    },
  )
}

const selectedReagentFlowRecord = computed<ReagentFlowRecord | undefined>(() => reagentFlowRecords.value.find((item) => item.recordNo === selectedReagentRecordNo.value) ?? reagentFlowRecords.value[0])

const reagentApprovalStatusText = (record: ReagentFlowRecord) => {
  return record.approvalStep ? '审批中' : record.approvalStatus ?? '未提交'
}

const reagentAddApprovalHistory = (record: ReagentFlowRecord, title: string) => {
  const user = currentLoginUser.value
  record.approvalHistory = [
    ...(record.approvalHistory ?? []),
    { title, user: user ? `${user.name}（${user.role}）` : '系统', time: currentDateTimeText() },
  ]
}

const reagentApprovalFlowItems = (record?: ReagentFlowRecord) => {
  if (!record) return []
  const history = record.approvalHistory ?? []
  return [
    { name: '提交留痕', state: history.some((item) => item.title.includes('提交')) || record.approvalStep || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: Send, desc: '过程记录提交' },
    { name: '编制人审批', state: record.approvalStep === '编制人' ? 'active' : history.some((item) => item.title.includes('编制人')) || ['审核人', '批准人'].includes(String(record.approvalStep)) || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: Edit3, desc: '留痕内容编制' },
    { name: '审核人审批', state: record.approvalStep === '审核人' ? 'active' : history.some((item) => item.title.includes('审核人')) || record.approvalStep === '批准人' || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: ClipboardList, desc: '过程复核' },
    { name: '批准人审批', state: record.approvalStep === '批准人' ? 'active' : record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: CheckCircle2, desc: '审批放行' },
    { name: '留痕归档', state: record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: PackageCheck, desc: '过程归档' },
  ]
}

const openReagentApprovalFlow = (recordNo: string) => {
  selectedReagentRecordNo.value = recordNo
  showReagentApprovalFlow.value = true
}

const canApproveReagentRecord = (record?: ReagentFlowRecord) => {
  if (!record) return false
  return record.approvalStep === currentLoginUser.value?.role
}

const currentReagentApprovalTodoRecords = computed(() => reagentFlowRecords.value.filter((record) => canApproveReagentRecord(record)))

const approveReagentRecord = (recordNo: string, action: 'pass' | 'return') => {
  selectedReagentRecordNo.value = recordNo
  const target = reagentFlowRecords.value.find((item) => item.recordNo === recordNo)
  if (!target || !canApproveReagentRecord(target)) return

  if (action === 'return') {
    reagentAddApprovalHistory(target, `${target.approvalStep}：不同意，退回修改`)
    target.approvalStatus = '退回修改'
    target.approvalStep = undefined
    return
  }

  if (target.approvalStep === '编制人') {
    reagentAddApprovalHistory(target, '编制人：审批通过/同意')
    target.approvalStep = '审核人'
    target.approvalStatus = '审批中'
  } else if (target.approvalStep === '审核人') {
    reagentAddApprovalHistory(target, '审核人：审批通过/同意')
    target.approvalStep = '批准人'
    target.approvalStatus = '审批中'
  } else if (target.approvalStep === '批准人') {
    reagentAddApprovalHistory(target, '批准人：审批通过/同意')
    target.approvalStep = undefined
    target.approvalStatus = '审批通过'
    if (target.action === '领用') {
      const stock = reagentStocks.value.find((item) => item.code === target.reagentCode)
      if (stock && !target.stockApplied) {
        stock.stock = Math.max(0, stock.stock - target.quantity)
        updateReagentStockStatus(stock)
        target.stockApplied = true
      }
      target.processStatus = '领用中'
    }
  }
}

const deleteSampleLabel = (sampleNo: string) => {
  if (!deletedLabelNos.value.includes(sampleNo)) {
    deletedLabelNos.value = [...deletedLabelNos.value, sampleNo]
  }
}

const formatDateTimeValue = (value: string) => value ? value.replace('T', ' ') : '2026-07-04 10:30'

const findSamplingRecordByOrderNo = (orderNo: string) => allSamplingRecords.value.find((item) => item.orderNo === orderNo)

const targetSamplingRecords = () => samplingSource.value === 'internal' ? internalSamplingRecords.value : externalSamplingRecords.value

const createSamplingForm = (overrides: Partial<SamplingForm> = {}): SamplingForm => {
  const cargoPosition = overrides.cargoPosition ? normalizeCargoPositionValue(overrides as SamplingRecord) : cargoPositionOptions[0].label
  return {
  orderNo: '',
  sampleNo: '',
  sampleName: '玉米质量检验样品',
  sampleCount: '2kg × 4',
  sampleUnit: 'kg',
  sampler: '张三',
  samplingDate: '2026-07-04',
  reason: samplingReason.value,
  grainType: '玉米',
  company: '中央储备粮镇江直属库有限公司',
  depot: '安鸿智慧粮库',
  representativeQuantity: '100.000',
  nature: '中央储备粮',
  origin: '江苏',
  productionYear: '2026',
  storageDate: '2026-06-17',
  packageType: '散装',
  retainCount: '1',
  inspectionCount: '1',
  keeper: '王保管',
  remark: '样品用于质量检验流程演示。',
  surveyNo: '',
  ...overrides,
  warehouseNo: cargoPositionWarehouseNo(cargoPosition),
  cargoPosition,
  }
}

const resetSamplingForm = () => {
  const sourcePrefix = samplingSource.value === 'internal' ? 'QY' : 'WQY'
  const samplePrefix = samplingSource.value === 'internal' ? 'YP' : 'WYP'
  const nextSeq = String(targetSamplingRecords().length + 1).padStart(3, '0')
  samplingReason.value = inspectionReasons[0]
  samplingForm.value = createSamplingForm({ orderNo: `${sourcePrefix}20260704${nextSeq}`, sampleNo: `${samplePrefix}20260704${nextSeq}`, reason: samplingReason.value })
}

const openSamplingForm = (mode: 'create' | 'edit', orderNo?: string) => {
  samplingFormMode.value = mode
  if (mode === 'edit' && orderNo) {
    const target = findSamplingRecordByOrderNo(orderNo)
    if (target) {
      samplingReason.value = target.reason
      samplingForm.value = createSamplingForm(target)
    }
  } else {
    resetSamplingForm()
  }
  showSamplingForm.value = true
}

const upsertSamplingRecord = (status: string) => {
  const records = targetSamplingRecords()
  const index = records.findIndex((item) => item.orderNo === samplingForm.value.orderNo)
  const cargoPosition = normalizeCargoPositionValue({ ...records[index], ...samplingForm.value } as SamplingRecord)
  const nextReason = activeQualityStandards({ grainType: samplingForm.value.grainType, reason: samplingReason.value }).length ? samplingReason.value : inspectionReasons[0]
  const nextRecord: SamplingRecord = { ...records[index], ...samplingForm.value, reason: nextReason, source: samplingSource.value === 'internal' ? '内部扦样' : '外部扦样', status, warehouseNo: cargoPositionWarehouseNo(cargoPosition), cargoPosition }
  if (index >= 0) {
    records[index] = nextRecord
  } else {
    records.unshift(nextRecord)
  }
  selectedSamplingNo.value = nextRecord.orderNo
  showSamplingForm.value = false
}

const deleteSamplingRecord = (orderNo: string) => {
  const records = targetSamplingRecords()
  const index = records.findIndex((item) => item.orderNo === orderNo)
  if (index >= 0) records.splice(index, 1)
}

const openSendSample = (orderNo: string) => {
  selectedSamplingNo.value = orderNo
  const target = findSamplingRecordByOrderNo(orderNo)
  sendSampleForm.value = { sender: target?.sender ?? currentLoginUser.value?.name ?? target?.sampler ?? '王帅', sendTime: (target?.sendTime ?? '2026-07-19 10:30').replace(' ', 'T') }
  showSendSample.value = true
}

const openReceiveSample = (orderNo: string) => {
  selectedSamplingNo.value = orderNo
  const target = findSamplingRecordByOrderNo(orderNo)
  receiveSampleForm.value = { receiver: target?.receiver ?? currentLoginUser.value?.name ?? '质检员A', receiveTime: (target?.receiveTime ?? '2026-07-19 11:10').replace(' ', 'T') }
  showReceiveSample.value = true
}

const confirmSendSample = () => {
  const target = findSamplingRecordByOrderNo(selectedSamplingNo.value)
  if (target) {
    target.status = '送样确认中'
    target.sender = sendSampleForm.value.sender
    target.sendTime = formatDateTimeValue(sendSampleForm.value.sendTime)
    target.samplingApprovalStatus = '审批中'
    target.samplingApprovalStep = '送样确认'
    target.samplingApprovalUser = sendSampleForm.value.sender
    target.samplingApprovalHistory = [
      ...(target.samplingApprovalHistory ?? []),
      { title: '提交送样确认', user: `${currentLoginUser.value?.name ?? target.sampler}（提交人）`, time: currentDateTimeText() },
    ]
  }
  showSendSample.value = false
}

const confirmReceiveSample = () => {
  const target = findSamplingRecordByOrderNo(selectedSamplingNo.value)
  if (target) {
    target.status = '收样确认中'
    target.receiver = receiveSampleForm.value.receiver
    target.receiveTime = formatDateTimeValue(receiveSampleForm.value.receiveTime)
    target.samplingApprovalStatus = '审批中'
    target.samplingApprovalStep = '收样确认'
    target.samplingApprovalUser = receiveSampleForm.value.receiver
    target.samplingApprovalHistory = [
      ...(target.samplingApprovalHistory ?? []),
      { title: '提交收样确认', user: `${currentLoginUser.value?.name ?? receiveSampleForm.value.receiver}（提交人）`, time: currentDateTimeText() },
    ]
  }
  showReceiveSample.value = false
}

const canApproveSamplingTransfer = (record?: SamplingRecord) => Boolean(record?.samplingApprovalStep && record.samplingApprovalUser === currentLoginUser.value?.name)

const samplingApprovalTodoRecords = computed(() => allSamplingRecords.value.filter((record) => canApproveSamplingTransfer(record)))

const openSamplingApprovalFlow = (record: SamplingRecord) => {
  selectedSamplingNo.value = record.orderNo
  samplingApprovalOpinion.value = ''
  showSamplingApprovalFlow.value = true
}

const approveSamplingTransfer = (action: 'pass' | 'return') => {
  const target = selectedSamplingRecord.value
  if (!target?.samplingApprovalStep || !canApproveSamplingTransfer(target)) return
  const step = target.samplingApprovalStep
  const user = currentLoginUser.value
  const resultTitle = action === 'pass' ? `${step}：审批通过并确认` : `${step}：退回修改`
  target.samplingApprovalHistory = [
    ...(target.samplingApprovalHistory ?? []),
    { title: resultTitle, user: user ? `${user.name}（确认人）` : '系统', time: currentDateTimeText(), ...(samplingApprovalOpinion.value.trim() ? { opinion: samplingApprovalOpinion.value.trim() } : {}) },
  ]
  if (action === 'return') {
    target.status = step === '送样确认' ? '扦样完成' : '已送样'
    target.samplingApprovalStatus = '退回修改'
  } else {
    target.status = step === '送样确认' ? '已送样' : '已收样'
    if (step === '收样确认') target.labelStatus = '待检'
    target.samplingApprovalStatus = '已确认'
  }
  target.samplingApprovalStep = undefined
  target.samplingApprovalUser = undefined
  showSamplingApprovalFlow.value = false
}

const closeSamplingForm = () => {
  showSamplingForm.value = false
}

const printPage = () => window.print()

const exportQualityReport = () => {
  const record = selectedReportRecord.value
  const escapeCsv = (value: string | number | undefined) => `"${String(value ?? '').replace(/"/g, '""')}"`
  const indicators = [...qualityLevelItems.value, ...storageQualityItems.value, ...healthIndicatorItems.value]
  const rows = [
    ['质量检验报告'],
    ['报告编号', record.reportNo ?? '未生成编号'],
    ['样品编号', displaySampleNo(record)],
    ['样品名称', record.sampleName],
    ['粮食品种', record.grainType],
    ['检验事由', record.reason],
    ['仓号', `${record.warehouseNo}仓`],
    [],
    ['检验项目', '指标属性', '检验依据', '标准值', '检验值', '单项判定', '检验人'],
    ...indicators.map((item) => [
      item.item,
      item.isKey ? '关键指标' : '普通指标',
      item.type,
      item.standard,
      qualityDetectValue(record, item.item) || healthDetectValue(record, item.item) || '—',
      qualityResultValue(record, item.item) || healthResultValue(record, item.item) || '—',
      record.reportResults?.[item.item]?.inspector ?? '质检员A',
    ]),
  ]
  const content = `\ufeff${rows.map((row) => row.map((item) => escapeCsv(item)).join(',')).join('\r\n')}`
  const blob = new Blob([content], { type: 'text/csv;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `${record.reportNo ?? displaySampleNo(record)}-质量检验报告.csv`
  link.click()
  URL.revokeObjectURL(url)
}

const openUserForm = (mode: 'create' | 'edit', user?: SystemUser) => {
  userFormMode.value = mode
  userForm.value = user ? { ...user } : { username: '', password: '123456', name: '', role: '质检员', department: '质量管理科', phone: '', status: '启用' }
  showUserForm.value = true
}

const saveUser = () => {
  if (!userForm.value.username || !userForm.value.name) return
  const index = systemUsers.value.findIndex((item) => item.username === userForm.value.username)
  if (index >= 0) {
    systemUsers.value[index] = { ...userForm.value }
  } else {
    systemUsers.value.push({ ...userForm.value })
  }
  showUserForm.value = false
}

const toggleUserStatus = (username: string) => {
  const target = systemUsers.value.find((item) => item.username === username)
  if (target) target.status = target.status === '启用' ? '停用' : '启用'
}

const deleteUser = (username: string) => {
  const index = systemUsers.value.findIndex((item) => item.username === username)
  if (index >= 0 && systemUsers.value.length > 1) systemUsers.value.splice(index, 1)
}

// The warehouse roofs form a staggered diagonal grid in the aerial image.
// Keeping the coordinates explicit makes future alignment against a new base map straightforward.
const warehouseMarkerCoordinates: WarehouseMarkerCoordinate[] = [
  { x: 38.0, y: 19.5 }, { x: 43.6, y: 23.6 }, { x: 48.7, y: 27.3 }, { x: 54.4, y: 31.8 },
  { x: 59.6, y: 35.2 }, { x: 66.2, y: 39.9 }, { x: 71.3, y: 43.7 }, { x: 77.5, y: 48.2 }, { x: 57.1, y: 41.2 },
  { x: 26.2, y: 31.6 }, { x: 32.6, y: 35.6 }, { x: 39.0, y: 39.6 }, { x: 45.4, y: 43.6 }, { x: 51.8, y: 47.6 }, { x: 58.2, y: 51.6 },
  { x: 20.9, y: 38.0 }, { x: 27.3, y: 42.0 }, { x: 33.7, y: 46.0 }, { x: 40.1, y: 50.0 }, { x: 46.5, y: 54.0 }, { x: 52.9, y: 58.0 }, { x: 59.3, y: 62.0 },
  { x: 17.2, y: 45.0 }, { x: 23.6, y: 49.0 }, { x: 30.0, y: 53.0 }, { x: 36.4, y: 57.0 }, { x: 42.8, y: 61.0 }, { x: 49.2, y: 65.0 }, { x: 55.6, y: 69.0 }, { x: 62.0, y: 73.0 },
]
const warehouseMapCalibrationStorageKey = 'quality-management-warehouse-map-calibration-v1'
const activeWarehouseMarkerCoordinates = ref<WarehouseMarkerCoordinate[]>(warehouseMarkerCoordinates.map((point) => ({ ...point })))

const warehouseMarkers = computed(() => activeWarehouseMarkerCoordinates.value.map((point, index) => {
  const warehouseNo = index + 49
  const stockQuantity = [7280, 8160, 8740, 7930, 8520][index % 5] + Math.floor(index / 5) * 120
  return {
    code: `${warehouseNo}仓`,
    ...point,
    grainType: warehouseNo % 2 === 0 ? '大豆' : '小麦',
    stockQuantity: `${stockQuantity.toLocaleString()} 吨`,
    qualityLevel: index % 4 === 1 ? '二等' : '一等',
    latestTestTime: `2026-06-${String(18 + (index % 8)).padStart(2, '0')}`,
  }
}))

const warehouseMapCalibrationSource = computed(() => {
  const coordinates = activeWarehouseMarkerCoordinates.value
    .map((point) => `  { x: ${point.x.toFixed(1)}, y: ${point.y.toFixed(1)} },`)
    .join('\n')
  return `const warehouseMarkerCoordinates: WarehouseMarkerCoordinate[] = [\n${coordinates}\n]`
})

const loadWarehouseMapCalibration = () => {
  const raw = localStorage.getItem(warehouseMapCalibrationStorageKey)
  if (!raw) return
  try {
    const coordinates = JSON.parse(raw) as WarehouseMarkerCoordinate[]
    if (!Array.isArray(coordinates) || !coordinates.length) return
    const validCoordinates = coordinates.filter((point) => Number.isFinite(point?.x) && Number.isFinite(point?.y))
    if (validCoordinates.length) {
      activeWarehouseMarkerCoordinates.value = validCoordinates.map((point) => ({
        x: Math.max(0, Math.min(100, Number(point.x.toFixed(1)))),
        y: Math.max(0, Math.min(100, Number(point.y.toFixed(1)))),
      }))
    }
  } catch {
    localStorage.removeItem(warehouseMapCalibrationStorageKey)
  }
}

const warehouseMapCoordinateFromPointer = (event: PointerEvent | MouseEvent) => {
  const rect = warehouseMapShell.value?.getBoundingClientRect()
  if (!rect) return null
  return {
    x: Math.max(0, Math.min(100, Number((((event.clientX - rect.left) / rect.width) * 100).toFixed(1)))),
    y: Math.max(0, Math.min(100, Number((((event.clientY - rect.top) / rect.height) * 100).toFixed(1)))),
  }
}

const updateWarehouseMapCalibrationPointer = (event: PointerEvent) => {
  if (!warehouseMapCalibrationMode.value) return
  const coordinate = warehouseMapCoordinateFromPointer(event)
  if (!coordinate) return
  warehouseMapCalibrationHover.value = coordinate
  const index = warehouseMapCalibrationDragIndex.value
  if (index !== null && activeWarehouseMarkerCoordinates.value[index]) {
    activeWarehouseMarkerCoordinates.value.splice(index, 1, coordinate)
  }
}

const handleWarehouseMapClick = (event: MouseEvent) => {
  if (!warehouseMapCalibrationMode.value) return
  const target = event.target as HTMLElement
  if (target.closest('.warehouse-marker, .warehouse-info-card, .map-legend, .map-calibration-tools, .map-calibration-panel')) return
  const coordinate = warehouseMapCoordinateFromPointer(event)
  if (!coordinate) return
  activeWarehouseMarkerCoordinates.value.push(coordinate)
  warehouseMapCalibrationSelectedIndex.value = activeWarehouseMarkerCoordinates.value.length - 1
}

const selectWarehouseMarker = (index: number) => {
  if (warehouseMapCalibrationMode.value) {
    warehouseMapCalibrationSelectedIndex.value = index
    return
  }
  selectedWarehouse.value = warehouseMarkers.value[index]?.code ?? selectedWarehouse.value
}

const startWarehouseMapCoordinateDrag = (index: number, event: PointerEvent) => {
  if (!warehouseMapCalibrationMode.value) return
  event.preventDefault()
  warehouseMapCalibrationSelectedIndex.value = index
  warehouseMapCalibrationDragIndex.value = index
  ;(event.currentTarget as HTMLElement).setPointerCapture(event.pointerId)
  updateWarehouseMapCalibrationPointer(event)
}

const finishWarehouseMapCoordinateDrag = () => {
  warehouseMapCalibrationDragIndex.value = null
}

const toggleWarehouseMapCalibration = () => {
  warehouseMapCalibrationMode.value = !warehouseMapCalibrationMode.value
  warehouseMapCalibrationHover.value = null
  warehouseMapCalibrationSelectedIndex.value = null
  finishWarehouseMapCoordinateDrag()
}

const deleteSelectedWarehouseMapCoordinate = () => {
  const index = warehouseMapCalibrationSelectedIndex.value
  if (index === null || activeWarehouseMarkerCoordinates.value.length <= 1) return
  activeWarehouseMarkerCoordinates.value.splice(index, 1)
  warehouseMapCalibrationSelectedIndex.value = null
}

const resetWarehouseMapCalibration = () => {
  activeWarehouseMarkerCoordinates.value = warehouseMarkerCoordinates.map((point) => ({ ...point }))
  warehouseMapCalibrationSelectedIndex.value = null
  warehouseMapCalibrationNotice.value = '已恢复源码点位'
}

const copyWarehouseMapCalibration = async () => {
  try {
    await navigator.clipboard.writeText(warehouseMapCalibrationSource.value)
    warehouseMapCalibrationNotice.value = '坐标已复制'
  } catch {
    warehouseMapCalibrationNotice.value = '复制失败，请检查浏览器权限'
  }
}

const clearWarehouseMapCalibrationHover = () => {
  if (warehouseMapCalibrationDragIndex.value === null) warehouseMapCalibrationHover.value = null
}

watch(activeWarehouseMarkerCoordinates, (coordinates) => {
  localStorage.setItem(warehouseMapCalibrationStorageKey, JSON.stringify(coordinates))
}, { deep: true })

const selectedWarehouseData = computed(() => {
  return warehouseMarkers.value.find((item) => item.code === selectedWarehouse.value) ?? warehouseMarkers.value[0]
})

const warehouseRiskPresentation = (record?: SamplingRecord) => {
  const risk = record ? qualityRiskLevel(record) : { code: 'normal' as QualityRiskCode, label: '正常' }
  if (risk.code === 'level1') return { status: 'alarm', qualityStatus: risk.label }
  if (risk.code === 'level2') return { status: 'warning', qualityStatus: risk.label }
  if (risk.code === 'level3') return { status: 'level3', qualityStatus: risk.label }
  return { status: 'normal', qualityStatus: risk.label }
}

const warehousePositionRows = (warehouse: typeof warehouseMarkers.value[number]) => {
  const warehouseNo = warehouse.code.replace('仓', '')
  const currentSample = currentWarehouseSampleRecordMap.value.get(warehouseNo)
  const presentation = warehouseRiskPresentation(currentSample)
  return [{
    position: `${warehouseNo}仓1号货位`,
    status: presentation.status,
    qualityStatus: presentation.qualityStatus,
    grainType: currentSample?.grainType ?? warehouse.grainType,
    stockQuantity: warehouse.stockQuantity,
    qualityLevel: currentSample ? reportQualityGrade(currentSample) : warehouse.qualityLevel,
    latestTestTime: currentSample?.receiveTime?.slice(0, 10) ?? currentSample?.samplingDate ?? warehouse.latestTestTime,
  }]
}

const warehouseMarkerStatus = (warehouse: typeof warehouseMarkers.value[number]) => {
  const rows = warehousePositionRows(warehouse)
  if (rows.some((item) => item.status === 'alarm')) return 'alarm'
  if (rows.some((item) => item.status === 'warning')) return 'warning'
  if (rows.some((item) => item.status === 'level3')) return 'level3'
  return 'normal'
}

const selectedWarehousePositions = computed(() => {
  return warehousePositionRows(selectedWarehouseData.value)
})

const selectedWarehousePositionData = computed(() => selectedWarehousePositions.value[activeWarehousePositionIndex.value % selectedWarehousePositions.value.length])

const warehouseRiskCount = computed(() => currentWarehouseSampleRecords.value.filter((record) => qualityRiskLevel(record).code !== 'normal').length)

const totalStock = computed(() => {
  return warehouseMarkers.value.reduce((total, item) => total + Number(item.stockQuantity.replace(/[^0-9]/g, '')), 0)
})

const overviewCards = computed(() => [
  { label: '库存总量', value: `${totalStock.value.toLocaleString()} 吨` },
  { label: '仓房总数', value: `43 座` },
  { label: '在储品种', value: '2 类' },
  { label: '质量合格率', value: '92.0%' },
  { label: '风险仓房', value: `${warehouseRiskCount.value} 座` },
  { label: '待检批次', value: '4 批' },
])

const processStatusText = (record: SamplingRecord) => {
  if (record.approvalStatus === '退回修改') return '退回修改'
  if (record.approvalStep) return '待审核'
  if (['留样', '销样'].includes(String(record.labelStatus)) || record.approvalStatus === '审批通过') return '已完成'
  if (record.status === '已收样' && ['待检', '在检', '检毕'].includes(String(record.labelStatus ?? '待检'))) return '检验中'
  if (record.status === '扦样完成') return '已扦样'
  if (record.status === '录入中') return '待扦样'
  return record.status
}

const isOverdueProcess = (record: SamplingRecord) => !['已收样'].includes(record.status) && record.samplingDate < '2026-07-04'

const flowStageSummary = (record: SamplingRecord) => {
  const hasReceived = record.status === '已收样'
  const labelStatus = hasReceived ? record.labelStatus ?? '待检' : '未收样'
  const reportStatus = record.reportNo ? `${record.reportNo}（${reportConclusion(record)}）` : labelStatus === '在检' ? '录入中' : labelStatus === '检毕' ? '待提交' : '未生成'
  const approvalStatus = record.approvalStep ? `${record.approvalStep}待审` : record.approvalStatus ?? '未提交'
  const ledgerStatus = hasReceived ? ledgerHandleMethod(record) : '未入账'
  return [
    `扦样单：${record.status}`,
    `样品标签：${labelStatus}`,
    `检验报告：${reportStatus}`,
    `在线审批：${approvalStatus}`,
    `样品台账：${ledgerStatus}`,
    `留样管理：${record.labelStatus === '留样' ? ledgerHandleTime(record) : '未留样'}`,
    `销样管理：${record.labelStatus === '销样' ? ledgerHandleTime(record) : '未销样'}`,
  ].join(' / ')
}

const taskStages = computed(() => [
  { label: '待扦样', value: allSamplingRecords.value.filter((item) => processStatusText(item) === '待扦样').length, icon: ClipboardPenLine },
  { label: '已扦样', value: allSamplingRecords.value.filter((item) => processStatusText(item) === '已扦样').length, icon: PackageCheck },
  { label: '检验中', value: allSamplingRecords.value.filter((item) => processStatusText(item) === '检验中').length, icon: Microscope },
  { label: '待审核', value: reportRecords.value.filter((item) => item.approvalStep).length, icon: ClipboardList },
  { label: '已完成', value: allSamplingRecords.value.filter((item) => processStatusText(item) === '已完成').length, icon: CheckCircle2 },
  { label: '超期', value: allSamplingRecords.value.filter((item) => isOverdueProcess(item)).length, icon: TriangleAlert },
])

const processDetails = computed(() => allSamplingRecords.value.map((record) => ({
  code: record.orderNo,
  warehouse: `${cargoPositionWarehouseNo(normalizeCargoPositionValue(record))}仓`,
  sample: displaySampleNo(record),
  status: isOverdueProcess(record) ? '超期' : processStatusText(record),
  grainType: record.grainType,
  owner: record.approvalStep ? systemUsers.value.find((user) => user.role === record.approvalStep)?.name ?? record.approvalStep : record.receiver ?? record.sender ?? record.sampler,
  taskTime: record.approvalHistory?.[record.approvalHistory.length - 1]?.time ?? record.receiveTime ?? record.sendTime ?? record.samplingDate,
  flowSummary: flowStageSummary(record),
})))

const shouldHideOwnerAndTime = (status: string) => status.startsWith('待')
const shouldHideGrainType = (status: string) => status === '待扦样'
</script>

<template>
  <div v-if="!currentLoginUser" class="login-page">
    <div class="login-logo">
      <img src="/中储粮logo.png" alt="中储粮" />
    </div>
    <div class="login-title">中储粮镇江直属库有限公司<br />&ensp;&ensp;&ensp;质量分析管理系统</div>
    <section class="login-card">
      <h1>用户登录</h1>
      <label class="login-field username-field"><input v-model="loginForm.username" placeholder="用户名/邮箱/手机号" /></label>
      <label class="login-field password-field"><input v-model="loginForm.password" type="password" placeholder="密码" /></label>
      <div v-if="loginError" class="unqualified-text">{{ loginError }}</div>
      <label class="remember-line"><input type="checkbox" style="zoom: 200%;" /> <span>记住密码</span></label>
      <button type="button" @click="handleLogin">登 录</button>
    </section>
    <footer>技术支持：杭州安鸿科技股份有限公司</footer>
  </div>

  <div v-else class="app-shell">
    <header class="top-bar">
      <div class="system-name">质量分析管理系统</div>
      <div class="top-user">
        <Menu :size="17" stroke-width="2.5" />
        <span>个人中心</span>
        <CircleUserRound class="avatar-icon" :size="19" />
        <span>{{ currentLoginUser.name }}</span>
        <button class="logout-button" type="button" @click="handleLogout">退出</button>
      </div>
    </header>

    <div class="app-body">
      <aside class="side-nav" aria-label="系统功能导航">
        <div class="menu-list">
          <button
            v-for="item in menuItems"
            :key="item.key"
            class="menu-item"
            :class="{ active: activeMenu === item.key }"
            type="button"
            @click="activeMenu = item.key"
          >
            <component :is="item.icon" :size="25" stroke-width="1.9" />
            <span>{{ item.label }}</span>
          </button>
        </div>
        <button class="collapse-btn" type="button" aria-label="收起菜单">
          <ChevronDown :size="25" stroke-width="2.2" />
        </button>
      </aside>

      <main class="workspace">
        <nav class="breadcrumb" aria-label="面包屑">
          <span>系统首页</span>
          <ChevronRight :size="13" />
          <span>{{ breadcrumbCurrent }}</span>
        </nav>

        <section v-if="activeMenu === 'dashboard'" class="dashboard-page">
          <div class="dashboard-grid">
            <section class="dashboard-panel overview-panel">
              <div class="panel-head">
                <h2>全库质量态势</h2>
              
              </div>

              <div class="overview-main-card">
                <div class="overview-label">库区在储情况</div>
                <div class="overview-empty">{{ totalStock.toLocaleString() }} 吨</div>
              
              </div>

              <div class="overview-card-grid">
                <div v-for="item in overviewCards" :key="item.label" class="overview-card">
                  <span>{{ item.label }}</span>
                  <strong>{{ item.value }}</strong>
                </div>
              </div>

              <div class="mini-chart-card">
                <div class="mini-chart-title">粮食品种占比</div>
                <div class="grain-share-card">
                  <div class="donut-chart" aria-label="粮食品种占比环形图"></div>
                  <div class="grain-share-legend">
                    <span><i class="wheat"></i>玉米 60%</span>
                    <span><i class="soybean"></i>大豆 40%</span>
                  </div>
                </div>
              </div>
            </section>

            <section class="dashboard-panel map-panel">
              <div class="panel-head map-head">
                <div>
                  <h2>仓房质量地图</h2>
                </div>
              </div>

              <div
                ref="warehouseMapShell"
                class="warehouse-map-shell"
                :class="{ calibrating: warehouseMapCalibrationMode }"
                @pointermove="updateWarehouseMapCalibrationPointer"
                @pointerleave="clearWarehouseMapCalibrationHover"
                @pointerup="finishWarehouseMapCoordinateDrag"
                @pointercancel="finishWarehouseMapCoordinateDrag"
                @click="handleWarehouseMapClick"
              >
                <div class="map-calibration-tools" @click.stop>
                  <button
                    type="button"
                    :class="{ active: warehouseMapCalibrationMode }"
                    :aria-pressed="warehouseMapCalibrationMode"
                    :title="warehouseMapCalibrationMode ? '关闭点位校准' : '开启点位校准'"
                    @click="toggleWarehouseMapCalibration"
                  >
                    <Crosshair :size="15" />
                  </button>
                </div>
                <div v-if="warehouseMapCalibrationMode" class="map-calibration-panel" @click.stop>
                  <div>
                    <span>{{ warehouseMapCalibrationHover ? `x ${warehouseMapCalibrationHover.x.toFixed(1)} / y ${warehouseMapCalibrationHover.y.toFixed(1)}` : 'x — / y —' }}</span>
                    <b>{{ activeWarehouseMarkerCoordinates.length }} 点</b>
                  </div>
                  <div class="map-calibration-actions">
                    <button type="button" title="复制当前点位坐标" @click="copyWarehouseMapCalibration"><Copy :size="14" /></button>
                    <button type="button" title="删除选中点位" :disabled="warehouseMapCalibrationSelectedIndex === null" @click="deleteSelectedWarehouseMapCoordinate"><Trash2 :size="14" /></button>
                    <button type="button" title="恢复源码默认点位" @click="resetWarehouseMapCalibration"><RotateCcw :size="14" /></button>
                  </div>
                  <small v-if="warehouseMapCalibrationNotice">{{ warehouseMapCalibrationNotice }}</small>
                </div>
                <img class="warehouse-map-image" src="/beijing.png" alt="镇江库区图片" />
                <button
                  v-for="(marker, index) in warehouseMarkers"
                  :key="marker.code"
                  class="warehouse-marker"
                  :class="[warehouseMarkerStatus(marker), { selected: !warehouseMapCalibrationMode && selectedWarehouse === marker.code, calibrating: warehouseMapCalibrationMode, 'calibration-selected': warehouseMapCalibrationMode && warehouseMapCalibrationSelectedIndex === index }]"
                  :style="{ left: `${marker.x}%`, top: `${marker.y}%` }"
                  type="button"
                  :title="warehouseMapCalibrationMode ? `${marker.code}：拖动微调` : marker.code"
                  @pointerdown.stop="startWarehouseMapCoordinateDrag(index, $event)"
                  @click.stop="selectWarehouseMarker(index)"
                >
                  <span>{{ marker.code }}</span>
                </button>
                <aside class="warehouse-info-card" aria-live="polite">
                  <div class="warehouse-info-head">
                    <div>
                      <strong>{{ selectedWarehouseData.code }}</strong>
                      <small>{{ selectedWarehousePositionData.position }}</small>
                    </div>
                    <span :class="['info-status', selectedWarehousePositionData.status]">
                      {{ selectedWarehousePositionData.qualityStatus }}
                    </span>
                  </div>
                  <div class="warehouse-info-grid">
                    <div>
                      <span>货位号</span>
                      <b>{{ selectedWarehousePositionData.position }}</b>
                    </div>
                    <div>
                      <span>粮食品种</span>
                      <b>{{ selectedWarehousePositionData.grainType }}</b>
                    </div>
                    <div>
                      <span>最近检验时间</span>
                      <b>{{ selectedWarehousePositionData.latestTestTime }}</b>
                    </div>
                    <div>
                      <span>库存数量</span>
                      <b>{{ selectedWarehousePositionData.stockQuantity }}</b>
                    </div>
                    <div>
                      <span>粮食质量等级</span>
                      <b>{{ selectedWarehousePositionData.qualityLevel }}</b>
                    </div>
                  </div>
                </aside>
                <div class="map-legend">
                  <span><i class="normal"></i>正常</span>
                  <span><i class="level3"></i>三级预警</span>
                  <span><i class="warning"></i>二级预警</span>
                  <span><i class="alarm"></i>一级预警</span>
                </div>
              </div>
            </section>

            <section class="dashboard-panel warning-panel">
              <div class="panel-head">
                <h2>预警风险概览</h2>
                <span>风险</span>
              </div>

              <div class="warning-total-card">
                <TriangleAlert :size="24" />
                <div>
                  <span>当前预警</span>
                  <strong>{{ currentWarningCount }}</strong>
                </div>
              </div>

              <div class="warning-donut-card">
                <div class="warning-donut" :style="warningDonutStyle"><span>{{ totalQualityRecords ? Math.round((currentWarningCount / totalQualityRecords) * 100) : 0 }}%</span></div>
                <div class="warning-donut-legend">
                  <span><i class="normal"></i>正常 {{ normalRate }}%</span>
                  <span><i class="level1"></i>一级 {{ level1Rate }}%</span>
                  <span><i class="level2"></i>二级 {{ level2Rate }}%</span>
                  <span><i class="level3"></i>三级 {{ level3Rate }}%</span>
                </div>
              </div>

            

              <div class="warning-summary-grid">
                <div><span>正常</span><b>{{ qualityWarningStats.normal }}</b></div>
                <div><span>预警类型</span><b>{{ warningTypes.length }}</b></div>
                <div><span>不合格项</span><b>{{ warningUnqualifiedTotal }}</b></div>
              </div>

           

              <div class="warning-list-placeholder">
                <div class="list-title">实时预警列表</div>
                <button
                  v-for="record in warningRecords"
                  :key="record.sampleNo"
                  type="button"
                  @click="openQualityWarningRecord(record.sampleNo, record.warehouse)"
                >
                  <span :class="['warning-tag', record.levelCode]">{{ record.level }}</span>
                  {{ record.position }} {{ record.sampleName }} {{ record.content }}
                </button>
                <button v-if="!warningRecords.length" type="button" disabled>
                  <span class="warning-tag normal">正常</span>
                  暂无质量预警记录
                  <em>已同步质量记录</em>
                </button>
              </div>
            </section>
          </div>

        </section>

        <div v-if="showProcessDialog" class="process-dialog-mask" @click.self="showProcessDialog = false">
          <section class="process-dialog" role="dialog" aria-modal="true" aria-label="检验流程跟踪详情">
            <div class="process-dialog-head">
              <div>
                <h2>检验流程跟踪</h2>
                <span>当前库区质量检验流程明细</span>
              </div>
              <button type="button" @click="showProcessDialog = false">关闭</button>
            </div>
            <div class="process-table-wrap">
              <table class="process-table">
                <thead>
                  <tr>
                    <th>任务编号</th>
                    <th>货位号</th>
                    <th>样品编号</th>
                    <th>流程状态</th>
                    <th>全流程信息</th>
                    <th>粮食品种</th>
                    <th>责任人</th>
                    <th>任务时间</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in processDetails" :key="item.code">
                    <td>{{ item.code }}</td>
                    <td>{{ item.warehouse }}</td>
                    <td>{{ item.sample }}</td>
                    <td><span class="process-status">{{ item.status }}</span></td>
                    <td>{{ item.flowSummary }}</td>
                    <td>{{ shouldHideGrainType(item.status) ? '—' : item.grainType }}</td>
                    <td>{{ shouldHideOwnerAndTime(item.status) ? '—' : item.owner }}</td>
                    <td>{{ shouldHideOwnerAndTime(item.status) ? '—' : item.taskTime }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </section>
        </div>

        <section v-else-if="activeMenu === 'inspection'" class="inspection-page">
          <div class="inspection-tabs">
            <button
              v-for="tab in inspectionTabs"
              :key="tab.key"
              type="button"
              :class="{ active: inspectionTab === tab.key }"
              @click="inspectionTab = tab.key"
            >
              {{ tab.label }}
            </button>
          </div>

          <div v-if="inspectionTab === 'overview'" class="inspection-overview">
            <div class="inspection-kpis">
              <button v-for="item in inspectionKpis" :key="item.label" :class="['inspection-kpi-card', item.tone]" type="button" @click="openInspectionTab(item.tab)">
                <span class="inspection-kpi-icon"><component :is="item.icon" :size="19" stroke-width="2" /></span><span class="inspection-kpi-content"><span>{{ item.label }}</span><strong>{{ item.value }}</strong></span>
              </button>
            </div>

            <div class="upcoming-business-card">
              <div class="panel-head"><div><h2>即将需要进行的业务</h2><span>优先处理当前队列中最紧急的一项业务</span></div><b>{{ upcomingBusinessTotal }} <small>项待办</small></b></div>
              <div class="upcoming-business-layout">
                <div class="upcoming-business-chart" aria-label="待办业务占比分布">
                  <svg viewBox="0 0 42 42" role="img" aria-label="待办业务占比环形图">
                    <circle class="upcoming-chart-track" cx="21" cy="21" r="15.9" />
                    <circle v-for="item in upcomingBusinessChartSegments" :key="item.key" :class="['upcoming-chart-segment', item.tone]" cx="21" cy="21" r="15.9" :stroke-dasharray="`${item.percent} ${100 - item.percent}`" :stroke-dashoffset="`${-item.offset}`" />
                  </svg>
                  <div><strong>{{ upcomingBusinessTotal }}</strong><span>待办总数</span></div>
                </div>
                <div class="upcoming-business-list">
                  <button v-for="item in upcomingBusinessItems" :key="item.key" type="button" :class="['upcoming-business-item', item.tone, { empty: !item.count }]" :disabled="!item.count" @click="openUpcomingBusiness(item)">
                    <span class="upcoming-business-icon"><component :is="item.icon" :size="17" stroke-width="2" /></span>
                    <span class="upcoming-business-content"><b>{{ item.title }}</b><small>{{ item.description }}</small></span>
                    <span class="upcoming-business-count"><strong>{{ item.count }}</strong><small>{{ item.count ? item.action : '暂无待办' }}</small></span>
                  </button>
                </div>
              </div>
            </div>

            <div class="overview-bottom-grid">
              <div class="flow-visual-card">
                <div class="panel-head"><h2>样品状态分布</h2><span>流转状态</span></div>
                <div class="sample-status-bars">
                  <div v-for="item in sampleStatusRows" :key="item.status"><span>{{ item.status }}</span><i :style="{ width: `${item.percent}%` }"></i><b>{{ item.count }}</b></div>
                </div>
              </div>
              <div class="flow-visual-card">
                <div class="panel-head"><h2>审批待办</h2><span>当前用户待处理</span></div>
                <div class="todo-list">
                  <button v-for="item in upcomingBusinessItems" :key="`todo-${item.key}`" type="button" :disabled="!item.count" @click="openUpcomingBusiness(item)"><component :is="item.icon" :size="14" /><span>{{ item.title }}</span><b>{{ item.count }}</b></button>
                </div>
              </div>
            </div>
          </div>

          <div v-else-if="inspectionTab === 'sampling'" class="sampling-page">
            <div class="sampling-toolbar">
              <div class="sampling-source-tabs">
                <button type="button" :class="{ active: samplingSource === 'internal' }" @click="samplingSource = 'internal'">内部扦样单</button>
                <button type="button" :class="{ active: samplingSource === 'external' }" @click="samplingSource = 'external'">外部扦样单</button>
              </div>
              <button class="primary-action" type="button" @click="openSamplingForm('create')"><Plus :size="14" /> 新增录入</button>
            </div>

            <div class="sampling-flow-summary">
              <div v-for="item in samplingFlowSummary" :key="item.label"><span>{{ item.label }}</span><strong>{{ item.value }}</strong></div>
            </div>

            <div class="sampling-filter-bar">
              <input v-model="samplingFilter.orderNo" placeholder="扦样单编号" />
              <input v-model="samplingFilter.sampleName" placeholder="样品名称" />
              <select v-model="samplingFilter.reason"><option value="">检验事由</option><option v-for="item in [...internalReasons, ...externalReasons].filter((value, index, array) => array.indexOf(value) === index)" :key="item">{{ item }}</option></select>
              <select v-model="samplingFilter.source"><option value="">样品来源</option><option>内部扦样</option><option>外部扦样</option></select>
              <select v-model="samplingFilter.status"><option value="">流程状态</option><option>录入中</option><option>扦样完成</option><option>送样确认中</option><option>已送样</option><option>收样确认中</option><option>已收样</option></select>
              <button type="button"><Search :size="14" /> 查询</button>
            </div>

            <div class="sampling-table-card">
              <table class="sampling-table sampling-order-table">
                <thead>
                  <tr>
                    <th>扦样单编号</th>
                    <th>样品编号</th>
                    <th>样品名称</th>
                    <th>货位</th>
                    <th>样品数量</th>
                    <th>代表样品数量</th>
                    <th>扦样人</th>
                    <th>扦样日期</th>
                    <th>样品来源</th>
                    <th>流程状态</th>
                    <th>检验事由</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="record in activeSamplingRecords" :key="record.orderNo">
                    <td>{{ record.orderNo }}</td>
                    <td>{{ displaySampleNo(record) }}</td>
                    <td>{{ record.sampleName }}</td>
                    <td>{{ normalizedCargoPosition(record) }}</td>
                    <td>{{ sampleCountWithUnit(record) }}</td>
                    <td>{{ record.representativeQuantity ?? '—' }}</td>
                    <td>{{ record.sampler }}</td>
                    <td>{{ record.samplingDate }}</td>
                    <td>{{ record.source }}</td>
                    <td><span class="sampling-status"><CheckCircle2 :size="12" /> {{ record.status }}</span></td>
                    <td>{{ record.reason }}</td>
                    <td>
                      <div class="sampling-actions">
                        <button v-if="record.status !== '录入中'" type="button" @click="openSamplingDetail(record.orderNo)"><Eye :size="13" /> 查看</button>
                        <button v-if="record.status === '录入中'" type="button" @click="openSamplingForm('edit', record.orderNo)"><Edit3 :size="13" /> 编辑</button>
                        <button v-if="record.status === '录入中'" type="button" @click="deleteSamplingRecord(record.orderNo)"><Trash2 :size="13" /> 删除</button>
                        <button v-if="record.status === '扦样完成'" type="button" @click="openSendSample(record.orderNo)"><Send :size="13" /> 送样</button>
                        <button v-if="record.status === '已送样'" type="button" @click="openReceiveSample(record.orderNo)"><PackageCheck :size="13" /> 收样</button>
                        <button v-if="record.status === '送样确认中' || record.status === '收样确认中'" type="button" @click="openSamplingApprovalFlow(record)"><Route :size="13" /> 审批流程</button>
                        <button v-if="record.samplingApprovalHistory?.length && !['送样确认中', '收样确认中'].includes(record.status)" type="button" @click="openSamplingApprovalFlow(record)"><Route :size="13" /> 审批流程</button>
                        <button v-if="canApproveSamplingTransfer(record)" type="button" class="sampling-confirm-action" @click="openSamplingApprovalFlow(record)"><CheckCircle2 :size="13" /> {{ record.samplingApprovalStep === '送样确认' ? '确认送样' : '确认收样' }}</button>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else-if="inspectionTab === 'label'" class="sampling-page">
            <div class="sampling-toolbar">
              <div class="module-title-block">
                <h2>样品标签管理</h2>
                <span>已收样样品自动进入标签管理，无需手动新增</span>
              </div>
            </div>

            <div class="sampling-filter-bar">
              <input v-model="labelFilter.sampleName" placeholder="样品名称" />
              <select v-model="labelFilter.source">
                <option value="">样品来源</option>
                <option>内部扦样</option>
                <option>外部扦样</option>
              </select>
              <input v-model="labelFilter.sampleNo" placeholder="样品编号" />
              <select v-model="labelFilter.printStatus">
                <option value="">打印状态</option>
                <option>未打印</option>
                <option>已打印</option>
              </select>
              <button type="button"><Search :size="14" /> 查询</button>
            </div>

            <div class="sampling-table-card">
              <table class="sampling-table label-table">
                <colgroup>
                  <col class="label-col-sample-no" />
                  <col class="label-col-name" />
                  <col class="label-col-quantity" />
                  <col class="label-col-representative" />
                  <col class="label-col-source" />
                  <col class="label-col-print-status" />
                  <col class="label-col-received-at" />
                  <col class="label-col-actions" />
                </colgroup>
                <thead>
                  <tr>
                    <th>样品编号</th>
                    <th>样品名称</th>
                    <th>样品数量</th>
                    <th>代表数量</th>
                    <th>样品来源</th>
                    <th>打印状态</th>
                    <th>来样时间</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="record in labelRecords" :key="record.sampleNo">
                    <td>{{ displaySampleNo(record) }}</td>
                    <td>{{ record.sampleName }}</td>
                    <td>{{ sampleCountWithUnit(record) }}</td>
                    <td>{{ record.representativeQuantity ?? '—' }}</td>
                    <td>{{ record.source }}</td>
                    <td><span :class="['label-print-status', labelPrintStatusText(record) === '已打印' ? 'printed' : 'unprinted']">{{ labelPrintStatusText(record) }}</span></td>
                    <td>{{ record.receiveTime }}</td>
                    <td>
                      <div class="sampling-actions">
                        <button type="button" @click="openSampleLabelPrint(record.sampleNo)"><Printer :size="13" /> {{ labelPrintStatusText(record) === '已打印' ? '再次打印' : '打印' }}</button>
                        <button type="button" @click="deleteSampleLabel(record.sampleNo)"><Trash2 :size="13" /> 删除</button>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else-if="inspectionTab === 'ledger'" class="ledger-page">
            <div class="ledger-head">
              <div class="ledger-title"><span></span><b>样品台账查询</b></div>
              <button class="ledger-print" type="button" @click="printPage"><Printer :size="13" /> 打印</button>
            </div>

            <div class="ledger-filter-panel">
              <label>样品品种
                <select v-model="ledgerFilter.grainType">
                  <option value="">请选择</option>
                  <option v-for="item in grainOptions" :key="item">{{ item }}</option>
                </select>
              </label>
              <label>扦样类型
                <select v-model="ledgerFilter.source">
                  <option value="">请选择</option>
                  <option>内部扦样</option>
                  <option>外部扦样</option>
                </select>
              </label>
              <label>检验事由
                <select v-model="ledgerFilter.reason">
                  <option value="">请选择</option>
                  <option v-for="item in [...internalReasons, ...externalReasons].filter((value, index, array) => array.indexOf(value) === index)" :key="item">{{ item }}</option>
                </select>
              </label>
              <label>接收日期
                <div class="ledger-date-range">
                  <input v-model="ledgerFilter.startDate" type="date" />
                  <span>-</span>
                  <input v-model="ledgerFilter.endDate" type="date" />
                </div>
              </label>
              <div class="ledger-filter-actions">
                <button type="button"><Search :size="13" /> 查询</button>
                <button type="button" @click="resetLedgerFilter">重置</button>
              </div>
            </div>

          

            <div class="ledger-sheet-wrap">
              <table class="ledger-sheet-table">
                <colgroup>
                  <col class="ledger-col-index" />
                  <col class="ledger-col-sample-no" />
                  <col class="ledger-col-grain" />
                  <col class="ledger-col-count" />
                  <col class="ledger-col-date" />
                  <col class="ledger-col-receiver" />
                  <col class="ledger-col-method" />
                  <col class="ledger-col-approver" />
                  <col class="ledger-col-reviewer" />
                  <col class="ledger-col-time" />
                  <col class="ledger-col-remark" />
                </colgroup>
                <caption>样品台账表</caption>
                <thead>
                  <tr>
                    <th rowspan="2">序号</th>
                    <th rowspan="2">样品编号</th>
                    <th rowspan="2">品种</th>
                    <th rowspan="2">样品数量</th>
                    <th rowspan="2">接收日期</th>
                    <th rowspan="2">收样人</th>
                    <th colspan="4">样品处理</th>
                    <th rowspan="2">备注</th>
                  </tr>
                  <tr>
                    <th>处理方式</th>
                    <th>批准人</th>
                    <th>审核人</th>
                    <th>处理时间</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(record, index) in ledgerRecords" :key="record.sampleNo">
                    <td>{{ index + 1 }}</td>
                    <td>{{ displaySampleNo(record) }}</td>
                    <td>{{ record.grainType }}</td>
                    <td>{{ sampleCountWithUnit(record) }}</td>
                    <td>{{ record.receiveTime?.slice(0, 10) ?? '—' }}</td>
                    <td class="signature-cell">{{ record.receiver ?? '—' }}</td>
                    <td>{{ ledgerHandleMethod(record) }}</td>
                    <td>{{ ledgerApproveUser(record, '批准人') }}</td>
                    <td>{{ ledgerApproveUser(record, '审核人') }}</td>
                    <td class="ledger-handle-time-cell">{{ ledgerHandleTime(record) }}</td>
                    <td>{{ record.source }} / {{ record.reason }}</td>
                  </tr>
                  <tr v-if="!ledgerRecords.length"><td colspan="11">暂无台账数据</td></tr>
                </tbody>
              </table>
            </div>

            <div class="ledger-pagination">
              <span>共 {{ ledgerRecords.length }} 条</span>
              <select><option>15条/页</option></select>
              <button type="button" disabled>‹</button>
              <button type="button" class="active">1</button>
              <button type="button" disabled>›</button>
              <span>前往</span><input value="1" /><span>页</span>
            </div>
          </div>

          <div v-else-if="inspectionTab === 'report'" class="sampling-page">
            <div class="sampling-toolbar">
              <div class="module-title-block">
                <h2>检验报告管理</h2>
                <span>扫码录入后填写质检结果，审批通过后生成正式质检报告</span>
              </div>
              <div class="approval-role-switch readonly">
                <span>当前登录用户</span>
                <b>{{ currentLoginUser.name }} / {{ currentLoginUser.role }}</b>
              </div>
            </div>

            <div class="sampling-filter-bar">
              <input v-model="reportFilters.sampleNo" placeholder="样品编号" />
              <input v-model="reportFilters.sampleName" placeholder="样品名称" />
              <select v-model="reportFilters.processStatus"><option value="">流程状态</option><option>待扫码录入</option><option>待录入结果</option><option>待修改结果</option><option>待提交审批</option><option>审核人审批中</option><option>负责人审批中</option><option>批准人审批中</option><option>质检报告已生成</option><option>审批未通过</option></select>
            </div>

            <div class="sampling-table-card">
              <table class="sampling-table">
                <thead>
                  <tr>
                    <th>样品编号</th>
                    <th>样品名称</th>
                    <th>样品数量</th>
                    <th>样品来源</th>
                    <th>流程状态</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="record in filteredReportRecords" :key="record.sampleNo">
                    <td>{{ record.sampleNo }}</td>
                    <td>{{ record.sampleName }}</td>
                    <td>{{ sampleCountWithUnit(record) }}</td>
                    <td>{{ record.source }}</td>
                    <td><span :class="['report-process-status', reportProcessStatusTone(record)]"><Route :size="13" /> {{ reportProcessStatusText(record) }}</span></td>
                    <td>
                      <div class="sampling-actions">
                        <button v-if="(record.labelStatus ?? '待检') === '待检'" type="button" @click="openInspectionScanDialog(record.sampleNo)"><Crosshair :size="13" /> 扫码录入</button>
                        <button v-if="canEditReportResult(record)" type="button" @click="openReportForm(record.sampleNo)"><Edit3 :size="13" /> 录入结果</button>
                        <button v-if="record.approvalStep || record.labelStatus === '检毕' || record.approvalStatus === '销样'" type="button" @click="openQualityResultView(record.sampleNo)"><Eye :size="13" /> 查看质检结果</button>
                        <button v-if="reportHasFormalReport(record)" type="button" @click="openReportView(record.sampleNo)"><BookOpenText :size="13" /> 查看质检报告</button>
                        <button v-if="record.approvalStep" type="button" @click="openApprovalFlow(record.sampleNo)"><Route :size="13" /> 流程图</button>
                        <button v-if="canApproveReport(record)" type="button" @click="approveReport(record.sampleNo, 'pass')"><CheckCircle2 :size="13" /> 同意</button>
                        <button v-if="canApproveReport(record)" type="button" @click="approveReport(record.sampleNo, 'return')"><CornerDownLeft :size="13" /> 退回修改</button>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else-if="inspectionTab === 'retain'" class="sampling-page">
            <div class="sampling-toolbar">
              <div class="module-title-block">
                <h2>样品留样管理</h2>
                <span>扫描样品标签建立留样记录，并通过独立审批流程完成确认</span>
              </div>
              <button class="primary-action" type="button" @click="openRetainScanDialog"><Crosshair :size="15" /> 扫码新增留样</button>
            </div>
            <div class="sampling-table-card">
              <table class="sampling-table retain-table">
                <thead><tr><th>留样编号</th><th>样品编号</th><th>样品名称</th><th>仓号/货位</th><th>来样日期</th><th>规格</th><th>留样数量</th><th>过期时间</th><th>样品/审批状态</th><th>处理方式</th><th>处置时间</th><th>操作</th></tr></thead>
                <tbody>
                  <tr v-for="record in retainRecords" :key="record.retainRecordId ?? record.sampleNo" :class="{ 'retain-approval-row': canApproveRetain(record) }">
                    <td>{{ record.retainRecordId ?? '历史留样' }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td><div class="retain-location-cell"><b>{{ record.warehouseNo }}仓</b><span>{{ (record.cargoPosition ?? `${record.warehouseNo}仓1号货位`).replace(`${record.warehouseNo}仓`, '') }}</span></div></td><td>{{ (record.receiveTime ?? record.samplingDate).slice(0, 10) }}</td><td>{{ record.retainSpecification ?? record.packageType ?? '—' }}</td><td>{{ retainCountWithUnit(record) }}</td><td><div :class="['retain-expire-progress', retainStatusLevel(record)]"><div class="retain-expire-progress-meta"><span>{{ retainExpireDate(record) }}</span><b>{{ retainExpireStatus(record) }}</b></div><div class="retain-expire-progress-track" role="progressbar" :aria-valuenow="retainExpireProgress(record)" aria-valuemin="0" aria-valuemax="100"><i :style="{ width: `${retainExpireProgress(record)}%` }"></i></div></div></td><td><div class="retain-state-stack"><span :class="['retain-status-badge', retainLifecycleTone(record)]"><PackageCheck :size="12" /> {{ retainLifecycleStatus(record) }}</span><span :class="['sampling-status', { 'retain-approval-reminder': canApproveRetain(record) }]">{{ canApproveRetain(record) ? `待您审批 · ${retainWorkflowStatusText(record)}` : retainWorkflowStatusText(record) }}</span></div></td><td>{{ record.retainHandlingMethod ?? '留存' }}</td><td>{{ record.retainDisposalTime || '—' }}</td>
                    <td><div class="sampling-actions retain-row-actions"><button v-if="record.retainRecordId" type="button" @click="openRetainApprovalFlow(record)"><Route :size="13" /> 审批流程</button><button v-if="record.retainApprovalStatus === '退回修改'" type="button" @click="openRetainEditForm(record)"><Edit3 :size="13" /> 修改并送审</button><button v-if="canStartDestroyFromRetain(record)" class="retain-destroy-action" type="button" @click="startDestroyFromRetain(record)"><Trash2 :size="13" /> 确认销样</button><button v-if="canApproveRetain(record)" class="retain-quick-approve" type="button" @click="approveRetainRecord('pass', record)"><CheckCircle2 :size="13" /> {{ record.retainApprovalStep === '确认留样' ? '确认留样' : '同意' }}</button><button v-if="canApproveRetain(record)" class="retain-quick-reject" type="button" @click="approveRetainRecord('return', record)"><CornerDownLeft :size="13" /> 拒绝</button></div></td>
                  </tr>
                  <tr v-if="!retainRecords.length"><td colspan="12">暂无留样数据，请在留样管理中独立登记。</td></tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else-if="inspectionTab === 'destroy'" class="sampling-page">
            <div class="sampling-toolbar">
              <div class="module-title-block">
                <h2>销样管理</h2>
                <span>选择已留样样品登记销样申请，并通过独立审批流程完成确认</span>
              </div>
              <button class="primary-action" type="button" @click="openDestroyForm()"><Plus :size="15" /> 新增销样登记</button>
            </div>
            <div class="sampling-table-card">
              <table class="sampling-table">
                <thead><tr><th>销样编号</th><th>样品编号</th><th>样品名称</th><th>仓号</th><th>货位号</th><th>规格</th><th>品种</th><th>销样来源</th><th>审批状态</th><th>操作</th></tr></thead>
                <tbody>
                  <tr v-for="record in destroyRecords" :key="record.destroyRecordId ?? record.sampleNo">
                    <td>{{ record.destroyRecordId ?? '历史销样' }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.warehouseNo }}仓</td><td>{{ (record.cargoPosition ?? `${record.warehouseNo}仓1号货位`).replace(`${record.warehouseNo}仓`, '') }}</td><td>{{ record.retainSpecification ?? record.packageType ?? record.sampleCount }}</td><td>{{ record.destroyVariety ?? record.grainType }}</td><td>{{ record.destroySource ?? (retainStatusLevel(record) === 'expired' ? '留样到期确认转入' : '历史销样') }}</td><td><span :class="['sampling-status', 'destroy-approval-status', destroyApprovalTone(record), { 'destroy-approval-current': canApproveDestroy(record) }]">{{ canApproveDestroy(record) ? `待您审批 · ${record.destroyApprovalStep}` : record.destroyApprovalStep ? `${record.destroyApprovalStep}审批中` : record.destroyApprovalStatus ?? '历史销样' }}</span></td>
                    <td><div class="sampling-actions destroy-row-actions"><button v-if="record.destroyRecordId" type="button" @click="openDestroyApprovalFlow(record)"><Route :size="13" /> 销样流程</button><button v-if="record.destroyApprovalStatus === '退回修改'" type="button" @click="openDestroyForm(record)"><Edit3 :size="13" /> 修改并送审</button><button v-if="canApproveDestroy(record)" class="retain-quick-approve" type="button" @click="approveDestroyRecord('pass', record)"><CheckCircle2 :size="13" /> {{ record.destroyApprovalStep === '确认销样' ? '确认销样' : '同意' }}</button><button v-if="canApproveDestroy(record)" class="retain-quick-reject" type="button" @click="approveDestroyRecord('return', record)"><CornerDownLeft :size="13" /> 拒绝</button><button v-if="!record.destroyRecordId" type="button" @click="openQualityResultView(record.sampleNo)"><Eye :size="13" /> 查看质检结果</button></div></td>
                  </tr>
                  <tr v-if="!destroyRecords.length"><td colspan="10">暂无销样数据，请先选择已留样样品新增销样登记。</td></tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else class="coming-panel">
            <h2>{{ inspectionTabs.find((item) => item.key === inspectionTab)?.label }}</h2>
            <p>该模块后续继续补充，目前先完成流程总览与扦样单管理。</p>
          </div>

          <div v-if="inspectionOverviewDrilldown" class="process-dialog-mask" @click.self="inspectionOverviewDrilldown = null">
            <section class="inspection-overview-dialog">
              <div class="process-dialog-head"><div><h2>{{ inspectionOverviewDrilldownMeta.title }}</h2><span>{{ inspectionOverviewDrilldownMeta.description }}</span></div><button type="button" @click="inspectionOverviewDrilldown = null"><X :size="12" /> 关闭</button></div>
              <div class="inspection-overview-dialog-body">
                <div class="overview-dialog-filters">
                  <input v-model="inspectionOverviewFilters.keyword" placeholder="样品编号、报告编号或名称" />
                  <label v-if="inspectionOverviewDrilldown === 'sampling'">扦样日期<input v-model="inspectionOverviewTodayDate" type="date" /></label>
                  <select v-model="inspectionOverviewFilters.warehouseNo"><option value="">全部仓号</option><option v-for="item in inspectionOverviewWarehouseOptions" :key="item" :value="item">{{ item }}仓</option></select>
                  <select v-if="['sampling', 'pending'].includes(inspectionOverviewDrilldown)" v-model="inspectionOverviewFilters.source"><option value="">全部来源</option><option>内部扦样</option><option>外部扦样</option></select>
                  <select v-if="['sampling', 'pending'].includes(inspectionOverviewDrilldown)" v-model="inspectionOverviewFilters.reason"><option value="">全部检验事由</option><option v-for="item in inspectionOverviewReasonOptions" :key="item" :value="item">{{ item }}</option></select>
                  <select v-if="inspectionOverviewDrilldown === 'retain'" v-model="inspectionOverviewFilters.retainStatus"><option value="">全部到期状态</option><option value="正常">未到期</option><option value="已到期">已到期</option></select>
                </div>

                <div class="overview-dialog-table-wrap">
                  <table v-if="inspectionOverviewDrilldown === 'sampling'" class="sampling-table"><thead><tr><th>扦样单号</th><th>样品编号</th><th>样品名称</th><th>扦样日期</th><th>扦样人</th><th>来源</th><th>检验事由</th><th>仓号</th><th>状态</th></tr></thead><tbody><tr v-for="record in inspectionOverviewRows" :key="record.orderNo"><td>{{ record.orderNo }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.samplingDate }}</td><td>{{ record.sampler }}</td><td>{{ record.source }}</td><td>{{ record.reason }}</td><td>{{ record.warehouseNo }}仓</td><td>{{ record.status }}</td></tr><tr v-if="!inspectionOverviewRows.length"><td colspan="9">暂无符合筛选条件的扦样单。</td></tr></tbody></table>

                  <table v-else-if="inspectionOverviewDrilldown === 'pending'" class="sampling-table"><thead><tr><th>报告编号</th><th>样品编号</th><th>样品名称</th><th>品种</th><th>检验事由</th><th>仓号</th><th>送样时间</th><th>报告状态</th></tr></thead><tbody><tr v-for="record in inspectionOverviewRows" :key="record.sampleNo"><td>{{ record.reportNo ?? '待生成' }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.grainType }}</td><td>{{ record.reason }}</td><td>{{ record.warehouseNo }}仓</td><td>{{ record.sendTime ?? '未送样' }}</td><td><span class="lab-status warn">待检</span></td></tr><tr v-if="!inspectionOverviewRows.length"><td colspan="8">暂无符合筛选条件的待检样品。</td></tr></tbody></table>

                  <table v-else-if="inspectionOverviewDrilldown === 'approval'" class="sampling-table"><thead><tr><th>报告编号</th><th>样品编号</th><th>样品名称</th><th>品种</th><th>检验事由</th><th>仓号</th><th>当前审批环节</th><th>审批状态</th></tr></thead><tbody><tr v-for="record in inspectionOverviewRows" :key="record.sampleNo"><td>{{ record.reportNo ?? '待生成' }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.grainType }}</td><td>{{ record.reason }}</td><td>{{ record.warehouseNo }}仓</td><td>{{ record.approvalStep }}</td><td><span class="lab-status warn">当前用户待审批</span></td></tr><tr v-if="!inspectionOverviewRows.length"><td colspan="8">当前用户暂无符合筛选条件的待审报告。</td></tr></tbody></table>

                  <table v-else-if="inspectionOverviewDrilldown === 'retain'" class="sampling-table"><thead><tr><th>报告编号</th><th>样品编号</th><th>样品名称</th><th>仓号</th><th>品种</th><th>留样数量</th><th>留样时间</th><th>过期时间</th><th>到期状态</th></tr></thead><tbody><tr v-for="record in inspectionOverviewRows" :key="record.sampleNo"><td>{{ record.reportNo ?? '—' }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.warehouseNo }}仓</td><td>{{ record.grainType }}</td><td>{{ retainCountWithUnit(record) }}</td><td>{{ record.retainTime ?? ledgerHandleTime(record) }}</td><td>{{ retainExpireDate(record) }}</td><td><span :class="['retain-expire-status', retainStatusLevel(record)]">{{ retainExpireStatus(record) }}</span></td></tr><tr v-if="!inspectionOverviewRows.length"><td colspan="9">暂无符合筛选条件的留样信息。</td></tr></tbody></table>

                  <table v-else class="sampling-table"><thead><tr><th>报告编号</th><th>样品编号</th><th>样品名称</th><th>仓号</th><th>品种</th><th>不合格项</th><th>销样来源</th><th>销样时间</th></tr></thead><tbody><tr v-for="record in inspectionOverviewRows" :key="record.sampleNo"><td>{{ record.reportNo ?? '—' }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.warehouseNo }}仓</td><td>{{ record.grainType }}</td><td>{{ reportUnqualifiedCount(record) }} 项</td><td>{{ retainStatusLevel(record) === 'expired' ? '确认销样后转入' : record.approvalStatus }}</td><td>{{ record.destroyTime ?? ledgerHandleTime(record) }}</td></tr><tr v-if="!inspectionOverviewRows.length"><td colspan="8">暂无符合筛选条件的销样信息。</td></tr></tbody></table>
                </div>
              </div>
            </section>
          </div>
        </section>

        <section v-else-if="activeMenu === 'warning'" class="inspection-page">
          <div class="warning-record-page">
            <div class="sampling-toolbar">
              <div class="module-title-block">
                <h2>质量预警记录</h2>
               
              </div>
            </div>

            <div class="warning-rule-cards">
              <div class="normal"><b>正常质量检验</b><strong>{{ qualityWarningStats.normal }} 条</strong><span>未触发质量预警</span></div>
              <div class="level1"><b>一级预警</b><strong>{{ qualityWarningStats.level1 }} 条</strong><span>严重质量风险，优先处置</span></div>
              <div class="level2"><b>二级预警</b><strong>{{ qualityWarningStats.level2 }} 条</strong><span>较高质量风险，及时复核</span></div>
              <div class="level3"><b>三级预警</b><strong>{{ qualityWarningStats.level3 }} 条</strong><span>一般质量风险，持续关注</span></div>
            </div>

            <div class="warning-filter-panel">
              <div class="warning-filter-tags" aria-label="预警等级筛选">
                <button v-for="tag in warningLevelTags" :key="tag.key" :class="['warning-filter-tag', tag.key, { active: warningLevelFilter === tag.key }]" type="button" @click="warningLevelFilter = tag.key"><span>{{ tag.label }}</span><b>{{ tag.count }}</b></button>
              </div>
              <div class="warning-filter-fields">
                <label class="warning-keyword-filter"><Search :size="14" /><input v-model="warningFilters.keyword" placeholder="样品编号、名称或报告编号" /></label>
                <select v-model="warningFilters.warehouseNo"><option value="">全部仓号</option><option v-for="warehouseNo in warningWarehouseOptions" :key="warehouseNo" :value="warehouseNo">{{ warehouseNo }}仓</option></select>
                <select v-model="warningFilters.reason"><option value="">全部检验事由</option><option v-for="reason in warningReasonOptions" :key="reason" :value="reason">{{ reason }}</option></select>
                <label class="warning-date-filter"><span>预警日期</span><input v-model="warningFilters.date" type="date" /></label>
                <button class="warning-filter-reset" type="button" @click="resetWarningFilters"><RotateCcw :size="14" /> 重置</button>
              </div>
            </div>

            <div class="sampling-table-card warning-record-table-card">
              <table class="sampling-table warning-record-table">
                <thead>
                  <tr>
                    <th>预警等级</th>
                    <th>报告编号</th>
                    <th>样品编号</th>
                    <th>样品名称</th>
                    <th>仓号</th>
                    <th>品种</th>
                    <th>检验事由</th>
                    <th>不合格项目数量</th>
                    <th>预警时间</th>
                    <th>判定说明</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in filteredQualityWarningRecords" :key="item.record.sampleNo">
                    <td><span :class="['warning-level-badge', item.risk.code === 'level1' ? 'alarm' : item.risk.code === 'level2' ? 'warning' : 'level3']">{{ item.level }}</span></td>
                    <td>{{ item.record.reportNo ?? '—' }}</td>
                    <td>{{ displaySampleNo(item.record) }}</td>
                    <td>{{ item.record.sampleName }}</td>
                    <td>{{ normalizedCargoPosition(item.record) }}</td>
                    <td>{{ item.record.grainType }}</td>
                    <td>{{ item.record.reason }}</td>
                    <td><button class="unqualified-count-btn" type="button" @click="openUnqualifiedItems(item.record.sampleNo)"><Eye :size="13" /> {{ item.count }} 项</button></td>
                    <td>{{ item.warningTime }}</td>
                    <td>{{ item.status }}</td>
                    <td><div class="sampling-actions"><button v-if="reportHasFormalReport(item.record)" type="button" @click="openReportView(item.record.sampleNo)"><BookOpenText :size="13" /> 查看质检报告</button><button v-else type="button" @click="openQualityResultView(item.record.sampleNo)"><Eye :size="13" /> 查看质检结果</button></div></td>
                  </tr>
                  <tr v-if="!filteredQualityWarningRecords.length"><td colspan="11">暂无符合筛选条件的质量预警记录。</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <section v-else-if="activeMenu === 'report'" class="inspection-page">
          <div class="quality-report-page">
            <div class="quality-report-hero">
              <div>
           
                <h2>质量报表管理</h2>
                
              </div>
              <div class="quality-report-metrics">
                <div><b>{{ completedReportRecords.length }}</b><span>报告样本</span></div>
                <div><b>{{ qualityWarningRecords.length }}</b><span>风险记录</span></div>
                <div><b>{{ warehouseSummaryRows.filter((item) => item.sampleCount > 0).length }}</b><span>涉及仓房</span></div>
              </div>
            </div>

            <div class="quality-report-tabs">
              <button v-for="tab in qualityReportTabs" :key="tab.key" :class="{ active: activeQualityReport === tab.key }" type="button" @click="activeQualityReport = tab.key">{{ tab.label }}</button>
            </div>

            <div class="quality-report-sheet" v-if="activeQualityReport === 'cargo'">
              <div class="report-sheet-head"><div><h3>质量分析货位明细表</h3><span>自动生成编号：ZL-HW-20260703</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <section class="warehouse-record-paper report-record-paper cargo-record-paper">
                <h2>中央储备粮镇江直属库有限公司质量分析货位明细表</h2>
                <div class="warehouse-record-meta">
                  <span>统计期间：2026-07-01 至 2026-07-03</span>
                  <span>制表部门：质量管理科</span>
                  <span>数据来源：检验流程管理系统</span>
                </div>
                <table class="warehouse-record-table cargo-record-table">
                  <colgroup>
                    <col class="cargo-col-sample" />
                    <col class="cargo-col-report" />
                    <col class="cargo-col-position" />
                    <col class="cargo-col-warehouse" />
                    <col class="cargo-col-grain" />
                    <col class="cargo-col-stock" />
                    <col class="cargo-col-count" />
                    <col class="cargo-col-reason" />
                    <col class="cargo-col-conclusion" />
                    <col class="cargo-col-risk" />
                  </colgroup>
                  <thead>
                    <tr class="warehouse-record-base">
                      <th>报告样本</th><td>{{ cargoDetailRows.length }}</td>
                      <th>涉及货位</th><td>{{ warehouseReportMeta.positions }}</td>
                      <th>涉及品种</th><td>{{ warehouseReportMeta.grainTypes }}</td>
                      <th>风险记录</th><td colspan="3">{{ qualityWarningRecords.length }}</td>
                    </tr>
                    <tr>
                      <th>样品编号</th>
                      <th>报告编号</th>
                      <th>货位</th>
                      <th>仓号</th>
                      <th>品种</th>
                      <th>库存数量</th>
                      <th>样品数量</th>
                      <th>检验事由</th>
                      <th>检验结论</th>
                      <th>风险项</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="row in cargoDetailRows" :key="row.sampleNo">
                      <td>{{ displaySampleNo(row) }}</td><td>{{ row.reportNo ?? '待生成' }}</td><td>{{ row.position }}</td><td>{{ row.warehouseNo }}仓</td><td>{{ row.grainType }}</td><td>{{ row.stock }}</td><td>{{ row.sampleCount }}</td><td>{{ row.reason }}</td><td>{{ row.conclusion }}</td><td>{{ row.unqualified.length ? row.unqualified.join('、') : '无' }}</td>
                    </tr>
                    <tr v-for="(_, index) in cargoReportBlankRows" :key="`cargo-blank-${index}`" class="warehouse-record-blank-row">
                      <td v-for="cell in 10" :key="cell"></td>
                    </tr>
                  </tbody>
                </table>
              </section>
            </div>

            <div class="quality-report-sheet" v-else-if="activeQualityReport === 'warehouse'">
              <div class="report-sheet-head"><div><h3>分货位质量汇总表</h3><span>自动生成编号：ZL-FHW-20260703</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <section class="warehouse-record-paper warehouse-summary-record-paper">
                <div class="warehouse-grain-tabs" role="tablist" aria-label="粮食品种">
                  <button
                    v-for="grain in warehouseReportGrainTabs"
                    :key="grain"
                    :class="{ active: activeWarehouseReportGrain === grain }"
                    type="button"
                    role="tab"
                    :aria-selected="activeWarehouseReportGrain === grain"
                    @click="activeWarehouseReportGrain = grain"
                  >{{ grain }}</button>
                </div>
                <h2>{{ activeWarehouseReportGrain }}分货位质量检测记录表</h2>
                <div class="warehouse-record-meta">
                  <span>直属企业：{{ warehouseReportMeta.enterprise }}</span>
                  <span>储粮性质：{{ warehouseReportMeta.nature }}</span>
                  <span>仓（货位）号：{{ warehouseReportMeta.positions }}</span>
                </div>
                <table class="warehouse-record-table warehouse-summary-record-table">
                  <colgroup>
                    <col class="warehouse-col-position" />
                    <col class="warehouse-col-date" />
                    <col v-for="indicator in activeWarehouseReportIndicators" :key="indicator.item" class="warehouse-col-indicator" />
                    <col class="warehouse-col-agency" />
                    <col class="warehouse-col-report" />
                    <col class="warehouse-col-recorder" />
                    <col class="warehouse-col-purpose" />
                    <col class="warehouse-col-remark" />
                  </colgroup>
                  <thead>
                    <tr class="warehouse-record-base">
                      <th>数量（吨）</th><td>{{ warehouseReportMeta.totalQuantity }}</td>
                      <th>等级</th><td>{{ warehouseReportMeta.grade }}</td>
                      <th>产地</th><td>{{ warehouseReportMeta.origin }}</td>
                      <th>适用标准</th><td :colspan="warehouseReportColumnCount - 7">{{ activeWarehouseReportGrain }}质量检验标准</td>
                    </tr>
                    <tr>
                      <th>仓（货位）号</th>
                      <th>检测日期</th>
                      <th v-for="indicator in activeWarehouseReportIndicators" :key="indicator.item">
                        {{ indicator.label }}<small>标准：{{ warehouseReportIndicatorStandard(indicator.item) }}</small>
                      </th>
                      <th>检测机构</th>
                      <th>质检报告编号</th>
                      <th>记录人</th>
                      <th>检测目的</th>
                      <th>备注</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="row in warehouseReportRows" :key="row.record.sampleNo">
                      <td>{{ row.position }}</td>
                      <td>{{ row.detectDate }}</td>
                      <td v-for="indicator in activeWarehouseReportIndicators" :key="indicator.item">{{ warehouseReportIndicatorValue(row.record, indicator.item) }}</td>
                      <td>{{ row.agency }}</td>
                      <td>{{ row.reportNo }}</td>
                      <td>{{ row.recorder }}</td>
                      <td>{{ row.purpose }}</td>
                      <td>{{ row.remark }}</td>
                    </tr>
                    <tr v-for="(_, index) in warehouseReportBlankRows" :key="`warehouse-blank-${index}`" class="warehouse-record-blank-row">
                      <td v-for="cell in warehouseReportColumnCount" :key="cell"></td>
                    </tr>
                  </tbody>
                </table>
              </section>
            </div>

            <div class="quality-report-sheet" v-else-if="activeQualityReport === 'acceptance'">
              <div class="report-sheet-head"><div><h3>验收申请函</h3><span>自动提取入库类检验样品生成验收申请明细</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <section class="warehouse-record-paper report-record-paper acceptance-record-paper">
                <h2>中央储备粮镇江直属库有限公司质量分析验收申请函</h2>
                <div class="warehouse-record-meta">
                  <span>编号：YS-SQ-20260703</span>
                  <span>申请部门：质量管理科</span>
                  <span>申请日期：2026-07-03</span>
                </div>
                <table class="warehouse-record-table acceptance-record-table">
                  <colgroup>
                    <col class="acceptance-col-report" />
                    <col class="acceptance-col-sample" />
                    <col class="acceptance-col-warehouse" />
                    <col class="acceptance-col-grain" />
                    <col class="acceptance-col-reason" />
                    <col class="acceptance-col-conclusion" />
                    <col class="acceptance-col-opinion" />
                  </colgroup>
                  <thead>
                    <tr class="warehouse-record-base">
                      <th>申请依据</th><td colspan="6">根据检验流程管理系统数据，以下入库相关样品已完成收样、检验及报告生成，现申请组织质量验收。</td>
                    </tr>
                    <tr>
                      <th>报告编号</th>
                      <th>样品编号</th>
                      <th>仓号</th>
                      <th>品种</th>
                      <th>检验事由</th>
                      <th>检验结论</th>
                      <th>申请意见</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="row in acceptanceRows" :key="row.sampleNo">
                      <td>{{ row.reportNo ?? '待生成' }}</td><td>{{ displaySampleNo(row) }}</td><td>{{ row.warehouseNo }}仓</td><td>{{ row.grainType }}</td><td>{{ row.reason }}</td><td>{{ reportConclusion(row) }}</td><td>{{ reportUnqualifiedCount(row) ? '建议复核后验收' : '建议通过验收' }}</td>
                    </tr>
                    <tr v-for="(_, index) in acceptanceReportBlankRows" :key="`acceptance-blank-${index}`" class="warehouse-record-blank-row">
                      <td v-for="cell in 7" :key="cell"></td>
                    </tr>
                  </tbody>
                </table>
              </section>
            </div>

            <div class="quality-report-sheet" v-else>
              <div class="report-sheet-head"><div><h3>春秋普检测结果汇总表</h3><span>自动汇总春季普查、秋季普查样品的检测结果</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <section class="warehouse-record-paper report-record-paper season-record-paper">
                <h2>春秋普检测结果汇总表</h2>
                <div class="warehouse-record-meta">
                  <span>统计期间：2026年度春秋普查</span>
                  <span>检测单位：中央储备粮镇江直属库有限公司检化验室</span>
               
                </div>
                <table class="warehouse-record-table season-record-table">
                  <colgroup>
                    <col class="season-col-sample" />
                    <col class="season-col-survey" />
                    <col class="season-col-type" />
                    <col class="season-col-warehouse" />
                    <col class="season-col-grain" />
                    <col class="season-col-value" />
                    <col class="season-col-acid" />
                    <col class="season-col-afb" />
                    <col class="season-col-health" />
                    <col class="season-col-conclusion" />
                  </colgroup>
                  <thead>
                    <tr class="warehouse-record-base">
                      <th>样品总数</th><td>{{ seasonRows.length }}</td>
                      <th>涉及仓号</th><td colspan="3">{{ [...new Set(seasonRows.map((row) => `${row.warehouseNo}仓`))].join('、') || '—' }}</td>
                      <th>涉及品种</th><td colspan="3">{{ [...new Set(seasonRows.map((row) => row.grainType))].join('、') || '—' }}</td>
                    </tr>
                    <tr>
                      <th>样品编号</th>
                      <th>普查编号</th>
                      <th>普查类型</th>
                      <th>仓号</th>
                      <th>品种</th>
                      <th>水分</th>
                      <th>酸价/脂肪酸值</th>
                      <th>黄曲霉毒素B1</th>
                      <th>卫生指标</th>
                      <th>综合判定</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="row in seasonRows" :key="row.sampleNo">
                      <td>{{ displaySampleNo(row) }}</td><td>{{ displaySurveyNo(row) }}</td><td>{{ row.reason }}</td><td>{{ row.warehouseNo }}仓</td><td>{{ row.grainType }}</td><td>{{ reportMeasureValue(row, ['水分 (g/100g)', '水分及挥发物(%)']) }}</td><td>{{ reportMeasureValue(row, ['粗脂肪酸值 (mg/g)', '脂肪酸值 (mgKOH/100g)', '酸价(KOH)/(mg/g)']) }}</td><td>{{ reportMeasureValue(row, ['黄曲霉毒素B1 (μg/kg)', '黄曲霉毒素B1(μg/kg)']) }}</td><td>{{ reportMeasureValue(row, ['铅 (mg/kg)', '铅(mg/kg)', '镉 (mg/kg)']) }}</td><td>{{ reportConclusion(row) }}</td>
                    </tr>
                    <tr v-for="(_, index) in seasonReportBlankRows" :key="`season-blank-${index}`" class="warehouse-record-blank-row">
                      <td v-for="cell in 10" :key="cell"></td>
                    </tr>
                  </tbody>
                </table>
              </section>
            </div>
          </div>
        </section>

        <section v-else-if="activeMenu === 'lab'" class="inspection-page">
          <div class="lab-page">
            <div class="lab-hero">
              <div><span>检化验室数字化</span><h2>检化验室管理</h2></div>
              <div class="lab-stat-grid"><div v-for="item in labStats" :key="item.label" :class="['lab-stat-card', item.tone]"><b>{{ item.value }}</b><span>{{ item.label }}</span></div></div>
            </div>

            <div class="inspection-tabs lab-module-tabs">
              <button type="button" :class="{ active: labTab === 'ledger' }" @click="labTab = 'ledger'">检化验仪器台账</button>
              <button type="button" :class="{ active: labTab === 'environment' }" @click="labTab = 'environment'">检化验室温湿度采集</button>
              <button type="button" :class="{ active: labTab === 'calibration' }" @click="labTab = 'calibration'">仪器检定校准记录</button>
              <button type="button" :class="{ active: labTab === 'maintenance' }" @click="labTab = 'maintenance'">仪器维护保养记录</button>
            </div>

            <div v-if="labTab === 'ledger'" class="lab-ledger-module">
              <section class="lab-panel instrument-panel">
                <div class="lab-panel-head"><div><h3>检化验仪器台账</h3><span>点击设备类别查看对应仪器明细</span></div><button type="button" @click="showInstrumentCategoryForm = true"><Plus :size="13" /> 新增设备类别</button></div>
                <div class="instrument-type-grid">
                  <button v-for="item in labInstrumentTypeSummary" :key="item.type" :class="['instrument-type-card', instrumentTypeTone(item.type), { active: activeInstrumentType === item.type }]" type="button" @click="activeInstrumentType = item.type">
                    <span class="instrument-type-icon"><component :is="instrumentTypeIcon(item.type)" :size="18" /></span>
                    <span class="instrument-type-name">{{ item.type }}</span>
                    <b>{{ item.count }}<small>台</small></b>
                    <div class="instrument-type-meta"><em>在用 {{ item.running }}</em><em :class="{ warn: item.warning > 0 }">异常 {{ item.warning }}</em></div>
                  </button>
                </div>
                <section class="custodian-instrument-section">
                  <div class="section-subhead"><div><h4>{{ activeInstrumentType }}仪器明细</h4><span>点击设备类别卡片切换 · 共 {{ activeInstrumentTypeRows.length }} 台</span></div><button type="button" class="secondary-action" @click="resetInstrumentEntryForm(); instrumentEntryForm.type = activeInstrumentType; showInstrumentEntryForm = true">新增仪器</button></div>
                  <div class="instrument-detail-scroll">
                    <table class="lab-table instrument-detail-table">
                      <colgroup><col class="instrument-code-col" /><col class="instrument-name-col" /><col class="instrument-model-col" /><col class="instrument-location-col" /><col class="instrument-custodian-col" /><col class="instrument-calibration-col" /><col class="instrument-progress-col" /><col class="instrument-status-col" /></colgroup>
                      <thead>
                        <tr>
                          <th>仪器编号</th>
                          <th>仪器名称</th>
                          <th>规格型号</th>
                          <th>位置</th>
                          <th>责任人</th>
                          <th>下次检定</th>
                          <th>检定进度</th>
                          <th>状态</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="item in activeInstrumentTypeRows" :key="item.code">
                          <td>{{ item.code }}</td>
                          <td>{{ item.name }}</td>
                          <td>{{ item.model }}</td>
                          <td>{{ item.location }}</td>
                          <td>{{ item.custodian }}</td>
                          <td><input :value="item.calibration" type="date" @change="updateInstrumentCalibration(item.code, ($event.target as HTMLInputElement).value)" /></td>
                          <td>
                            <div class="calibration-progress compact">
                              <div><span>{{ calibrationStatus(item).label }}</span><em :class="calibrationStatus(item).tone">{{ calibrationStatus(item).tone === 'ok' ? '正常' : calibrationStatus(item).tone === 'warn' ? '临期' : '过期' }}</em></div>
                              <i><b :class="calibrationStatus(item).tone" :style="{ width: `${calibrationProgress(item)}%` }"></b></i>
                            </div>
                          </td>
                          <td><span :class="['lab-status', instrumentStatusTone(item)]">{{ item.status }}</span></td>
                        </tr>
                        <tr v-if="!activeInstrumentTypeRows.length"><td colspan="8">该设备类别下暂无仪器，请新增仪器台账。</td></tr>
                      </tbody>
                    </table>
                  </div>
                </section>
              </section>

            </div>

            <section v-else-if="labTab === 'environment'" class="lab-panel lab-single-module env-panel environment-module">
                <div class="lab-panel-head"><div><h3>检化验室温湿度采集</h3></div><button type="button" @click="showEnvironmentTrendDialog = true"><ChartNoAxesCombined :size="14" /> 查看趋势图</button></div>
                <section class="environment-live-section">
                  <div class="environment-live-head"><div><i></i><b>实时温湿度</b></div><div class="environment-live-summary"><span>平均温度 <b>{{ environmentAvgTemperature }}</b></span><span>平均湿度 <b>{{ environmentAvgHumidity }}</b></span><span>异常 <b>{{ environmentWarningCount }}</b></span></div></div>
                  <div class="environment-live-grid">
                    <div v-for="item in realtimeEnvironmentRecords" :key="item.area" class="environment-live-item"><div><b>{{ item.area }}</b><span>{{ item.time }}</span></div><strong><small>温度</small>{{ item.temperature }}℃</strong><strong><small>湿度</small>{{ item.humidity }}%</strong><em :class="['lab-status', environmentStatusTone(item.status)]">{{ item.status }}</em></div>
                  </div>
                </section>
                <div class="environment-detail-toolbar">
                  <div class="env-range-tabs"><button v-for="range in environmentRanges" :key="range.key" type="button" :class="{ active: activeEnvironmentRange === range.key }" @click="activeEnvironmentRange = range.key">{{ range.label }}</button></div>
                  <label>区域<select v-model="activeEnvironmentArea"><option>全部区域</option><option v-for="area in environmentAreas" :key="area">{{ area }}</option></select></label>
                  <label>状态<select v-model="activeEnvironmentAlarmType"><option v-for="item in environmentAlarmTypes" :key="item.key" :value="item.key">{{ item.label }}</option></select></label>
                </div>
                <div class="environment-detail-head"><div><h4>{{ activeEnvironmentRangeLabel }}采集明细</h4><span>{{ activeEnvironmentArea }} · {{ environmentAlarmTypes.find((item) => item.key === activeEnvironmentAlarmType)?.label }}</span></div><b>{{ environmentDisplayRecords.length }} 条</b></div>
                <div class="environment-detail-table-wrap"><table class="lab-table environment-detail-table"><colgroup><col class="environment-time-col" /><col class="environment-area-col" /><col class="environment-value-col" /><col class="environment-value-col" /><col class="environment-status-col" /></colgroup><thead><tr><th>采集时间</th><th>采集区域</th><th>温度</th><th>湿度</th><th>状态</th></tr></thead><tbody><tr v-for="item in environmentDisplayRecords" :key="`${item.area}-${item.time}`"><td>{{ item.time }}</td><td>{{ item.area }}</td><td><b>{{ item.temperature }}℃</b></td><td><b>{{ item.humidity }}%</b></td><td><span :class="['lab-status', environmentStatusTone(item.status)]">{{ item.status }}</span></td></tr><tr v-if="!environmentDisplayRecords.length"><td colspan="5">当前筛选条件下暂无温湿度采集记录。</td></tr></tbody></table></div>
              </section>

            <section v-else-if="labTab === 'calibration'" class="lab-panel lab-single-module calibration-module">
              <div class="lab-panel-head"><div><h3>仪器检定校准记录</h3><span>由仪器台账自动生成检定计划，并沉淀每台设备的检定历史</span></div></div>
              <div class="calibration-summary-grid">
                <div v-for="item in calibrationStats" :key="item.label" :class="['calibration-summary-card', item.tone]"><b>{{ item.value }}</b><span>{{ item.label }}</span></div>
              </div>
              <div class="calibration-layout">
                <section class="calibration-plan-panel">
                  <div class="section-subhead"><div><h4>检定计划</h4><span>点击仪器查看对应检定历史</span></div><span>{{ calibrationPlanRows.length }} 台</span></div>
                  <div class="calibration-scroll">
                    <table class="lab-table calibration-plan-table">
                      <thead>
                        <tr>
                          <th>仪器名称</th>
                          <th>仪器编号</th>
                          <th>设备类别</th>
                          <th>下次检定</th>
                          <th>剩余天数</th>
                          <th>进度</th>
                          <th>状态</th>
                          <th>历史</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="item in calibrationPlanRows" :key="item.code" :class="{ active: selectedCalibrationInstrumentCode === item.code }" @click="openCalibrationHistory(item.code)">
                          <td>{{ item.instrument }}</td>
                          <td>{{ item.code }}</td>
                          <td>{{ item.type }}</td>
                          <td><input :value="item.planDate" type="date" @click.stop @change="updateInstrumentCalibration(item.code, ($event.target as HTMLInputElement).value)" /></td>
                          <td>{{ item.remainingDays > 0 ? `${item.remainingDays} 天后` : item.remainingDays === 0 ? '今天到期' : `已过期 ${Math.abs(item.remainingDays)} 天` }}</td>
                          <td>
                            <div class="calibration-progress compact">
                              <div><span>{{ item.statusLabel }}</span><em :class="item.statusTone">{{ item.statusTone === 'ok' ? '正常' : item.statusTone === 'warn' ? '临期' : '过期' }}</em></div>
                              <i><b :class="item.statusTone" :style="{ width: `${item.progress}%` }"></b></i>
                            </div>
                          </td>
                          <td><span :class="['lab-status', item.statusTone]">{{ item.statusLabel }}</span></td>
                          <td><button type="button" class="secondary-link" @click.stop="openCalibrationHistory(item.code)">查看</button></td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </section>

              </div>
            </section>

            <section v-else-if="labTab === 'maintenance'" class="lab-panel lab-single-module">
              <div class="lab-panel-head"><div><h3>仪器维护保养记录</h3><span>清洁、维护、故障处理和保养计划闭环管理</span></div></div>
              <div class="maintenance-scroll">
                <table class="lab-table maintenance-table">
                  <thead><tr><th>仪器名称</th><th>仪器编号</th><th>上次维护</th><th>下次维护</th><th>维护类型</th><th>处理人</th><th>剩余天数</th><th>维护进度</th><th>状态</th><th>操作</th></tr></thead>
                  <tbody>
                    <tr v-for="item in maintenanceRecords" :key="item.code">
                      <td>{{ item.instrument }}</td>
                      <td>{{ item.code }}</td>
                      <td>{{ item.lastDate }}</td>
                      <td>{{ item.nextDate }}</td>
                      <td>{{ item.type }}</td>
                      <td>{{ item.handler }}</td>
                      <td>{{ item.remainingDays > 0 ? `${item.remainingDays} 天后` : item.remainingDays === 0 ? '今天到期' : `已逾期 ${Math.abs(item.remainingDays)} 天` }}</td>
                      <td>
                        <div class="calibration-progress compact">
                          <div><span>{{ item.statusLabel }}</span></div>
                          <i><b :class="item.statusTone" :style="{ width: `${item.progress}%` }"></b></i>
                        </div>
                      </td>
                      <td><span :class="['lab-status', item.statusTone]">{{ item.statusLabel }}</span></td>
                      <td>
                        <div class="maintenance-actions">
                          <button v-if="item.canMaintain" type="button" class="secondary-link" @click="openMaintenanceForm(labInstruments.find((instrument) => instrument.code === item.code)!)">维护</button>
                          <span v-else class="maintenance-empty">{{ item.isAbandoned ? '已弃用' : '—' }}</span>
                          <button type="button" class="secondary-link abandon-link" :disabled="item.isAbandoned" @click="openAbandonForm(labInstruments.find((instrument) => instrument.code === item.code)!)">弃用</button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </section>
          </div>

          <div v-if="showCalibrationHistoryDialog && selectedCalibrationInstrument" class="process-dialog-mask" @click.self="showCalibrationHistoryDialog = false">
            <section class="calibration-history-dialog">
              <div class="process-dialog-head"><div><h2>仪器检定校准历史</h2><span>{{ selectedCalibrationInstrument.name }} · {{ selectedCalibrationInstrument.code }}</span></div><button type="button" @click="showCalibrationHistoryDialog = false"><X :size="14" /> 关闭</button></div>
              <div class="calibration-history-dialog-body">
                <div class="calibration-detail-card">
                  <div><b>{{ selectedCalibrationInstrument.name }}</b><span>{{ selectedCalibrationInstrument.type }}</span></div>
                  <div class="calibration-detail-grid">
                    <div><span>仪器编号</span><b>{{ selectedCalibrationInstrument.code }}</b></div>
                    <div><span>当前位置</span><b>{{ selectedCalibrationInstrument.location }}</b></div>
                    <div><span>责任人</span><b>{{ selectedCalibrationInstrument.custodian }}</b></div>
                    <div><span>当前下次检定</span><b>{{ selectedCalibrationInstrument.calibration }}</b></div>
                  </div>
                </div>
                <div class="calibration-history-dialog-head"><div><h3>检定历史记录</h3><span>当前计划与历次检定结果</span></div><b>{{ selectedCalibrationHistoryRows.length }} 条</b></div>
                <div class="calibration-history-list">
                  <article v-for="record in selectedCalibrationHistoryRows" :key="record.recordNo" :class="['calibration-history-item', { current: record.recordNo.endsWith('-CURRENT') }]">
                    <div class="calibration-history-main"><b>{{ record.actualDate }}</b><span>计划 {{ record.planDate }}</span></div>
                    <div class="calibration-history-meta"><span>{{ record.agency }}</span><em>{{ record.result }}</em></div>
                    <strong>{{ record.certificate }}</strong>
                  </article>
                </div>
              </div>
            </section>
          </div>

          <div v-if="showEnvironmentTrendDialog" class="process-dialog-mask" @click.self="showEnvironmentTrendDialog = false">
            <section class="environment-trend-dialog">
              <div class="process-dialog-head"><div><h2>检化验室温湿度趋势</h2><span>{{ activeEnvironmentRangeLabel }} · {{ activeEnvironmentArea }}</span></div><button type="button" @click="showEnvironmentTrendDialog = false"><X :size="14" /> 关闭</button></div>
              <div class="environment-trend-dialog-body">
                <div class="environment-trend-toolbar"><div class="env-range-tabs"><button v-for="range in environmentRanges" :key="range.key" type="button" :class="{ active: activeEnvironmentRange === range.key }" @click="activeEnvironmentRange = range.key">{{ range.label }}</button></div><label>区域<select v-model="activeEnvironmentArea"><option>全部区域</option><option v-for="area in environmentAreas" :key="area">{{ area }}</option></select></label></div>
                <div class="environment-trend-summary"><span>统计维度 <b>{{ activeEnvironmentRangeLabel }}</b></span><span>平均温度 <b>{{ environmentTrendAvgTemperature }}</b></span><span>平均湿度 <b>{{ environmentTrendAvgHumidity }}</b></span></div>
                <div class="env-trend-chart environment-popup-chart">
                  <div class="chart-legend"><span class="temp">温度</span><span class="humidity">湿度</span></div>
                  <div v-if="hoveredEnvironmentPoint" class="chart-tooltip">{{ hoveredEnvironmentPoint.label }} · {{ formatEnvironmentTooltipDate(activeEnvironmentRange, hoveredEnvironmentPoint.date) }} · 温度 {{ hoveredEnvironmentPoint.temperature }}℃ · 湿度 {{ hoveredEnvironmentPoint.humidity }}%</div>
                  <svg :viewBox="`0 0 ${environmentChartWidth} ${environmentChartHeight}`" role="img" aria-label="检化验室温湿度趋势图">
                    <line :x1="environmentChartPaddingX" :y1="environmentChartPaddingY" :x2="environmentChartPaddingX" :y2="environmentChartLabelY - 30" />
                    <line :x1="environmentChartPaddingX" :y1="environmentChartLabelY - 30" :x2="environmentChartWidth - environmentChartPaddingX" :y2="environmentChartLabelY - 30" />
                    <template v-for="(point, index) in environmentChartRows" :key="`${point.date}-${point.label}-axis`"><line class="chart-x-guide" :x1="point.x" :y1="environmentChartPaddingY" :x2="point.x" :y2="environmentChartLabelY - 30" /><text v-if="showEnvironmentAxisLabel(index, environmentChartRows.length)" class="chart-axis-text" :x="point.x" :y="environmentChartLabelY" text-anchor="middle">{{ point.label }}</text></template>
                    <polyline class="temperature-line" :points="environmentTemperaturePoints" /><polyline class="humidity-line" :points="environmentHumidityPoints" />
                    <template v-for="point in environmentChartRows" :key="`${point.date}-${point.label}`"><circle class="temperature-dot" r="4" v-bind="environmentPointPosition(point, 'temperature')" @mouseenter="hoveredEnvironmentPoint = point" @mouseleave="hoveredEnvironmentPoint = null" /><circle class="humidity-dot" r="4" v-bind="environmentPointPosition(point, 'humidity')" @mouseenter="hoveredEnvironmentPoint = point" @mouseleave="hoveredEnvironmentPoint = null" /></template>
                  </svg>
                </div>
              </div>
            </section>
          </div>

          <div v-if="showMaintenanceForm" class="process-dialog-mask" @click.self="showMaintenanceForm = false">
            <section class="send-sample-dialog maintenance-dialog">
              <div class="process-dialog-head"><div><h2>登记维护保养</h2><span>{{ labInstruments.find((item) => item.code === selectedMaintenanceInstrumentCode)?.name }} · {{ selectedMaintenanceInstrumentCode }}</span></div><button type="button" @click="showMaintenanceForm = false"><X :size="14" /> 关闭</button></div>
              <div class="send-form">
                <label>本次维护日期<input v-model="maintenanceForm.date" type="date" /></label>
                <label>维护类型<select v-model="maintenanceForm.type"><option>月度维护</option><option>清洁保养</option><option>功能校验</option><option>故障处理</option></select></label>
                <label>处理人<input v-model="maintenanceForm.handler" placeholder="请输入处理人" /></label>
                <label>下次维护日期<input v-model="maintenanceForm.nextDate" type="date" /></label>
                <button type="button" @click="saveMaintenanceRecord">确认维护</button>
              </div>
            </section>
          </div>

          <div v-if="showAbandonForm" class="process-dialog-mask" @click.self="showAbandonForm = false">
            <section class="send-sample-dialog maintenance-dialog">
              <div class="process-dialog-head"><div><h2>登记仪器弃用</h2><span>{{ labInstruments.find((item) => item.code === selectedAbandonInstrumentCode)?.name }} · {{ selectedAbandonInstrumentCode }}</span></div><button type="button" @click="showAbandonForm = false"><X :size="14" /> 关闭</button></div>
              <div class="send-form">
                <label>弃用时间<input v-model="abandonForm.date" type="date" /></label>
                <label>处理人<input v-model="abandonForm.handler" placeholder="请输入处理人" /></label>
                <button type="button" class="abandon-confirm" @click="saveAbandonRecord">确认弃用</button>
              </div>
            </section>
          </div>

          <div v-if="showInstrumentCategoryForm" class="process-dialog-mask" @click.self="showInstrumentCategoryForm = false">
            <section class="send-sample-dialog">
              <div class="process-dialog-head"><div><h2>新增设备类别</h2><span>填写设备类别、用途和库区信息</span></div><button type="button" @click="showInstrumentCategoryForm = false"><X :size="14" /> 关闭</button></div>
              <div class="send-form">
                <label>设备类别名称<input v-model="instrumentCategoryForm.type" placeholder="如：环境控制设备" /></label>
                <label>类别说明<input v-model="instrumentCategoryForm.description" placeholder="如：恒温、恒湿及环境监测设备" /></label>
                <label>默认库区<input v-model="instrumentCategoryForm.storageArea" placeholder="如：检化验室A区" /></label>
                <label class="full">备注<textarea v-model="instrumentCategoryForm.note" placeholder="填写类别备注或管理说明"></textarea></label>
                <button type="button" @click="addInstrumentCategory">确认新增</button>
              </div>
            </section>
          </div>

          <div v-if="showInstrumentEntryForm" class="process-dialog-mask" @click.self="showInstrumentEntryForm = false">
            <section class="send-sample-dialog">
              <div class="process-dialog-head"><div><h2>新增仪器</h2><span>填写仪器名称、规格型号、位置、责任人和检定日期</span></div><button type="button" @click="showInstrumentEntryForm = false"><X :size="14" /> 关闭</button></div>
              <div class="send-form">
                <label>设备类别<select v-model="instrumentEntryForm.type"><option v-for="type in instrumentTypeOptions" :key="type" :value="type">{{ type }}</option></select></label>
                <label>仪器名称<input v-model="instrumentEntryForm.name" placeholder="如：谷物水分测定仪" /></label>
                <label>规格型号<input v-model="instrumentEntryForm.model" placeholder="如：LDS-1G" /></label>
                <label>位置<input v-model="instrumentEntryForm.location" placeholder="如：检化验室A区" /></label>
                <label>责任人<input v-model="instrumentEntryForm.custodian" placeholder="如：质检员A" /></label>
                <label>检定日期<input v-model="instrumentEntryForm.calibration" type="date" /></label>
                <button type="button" @click="addInstrument">确认新增</button>
              </div>
            </section>
          </div>
        </section>

        <section v-else-if="activeMenu === 'reagent'" class="inspection-page">
          <div class="reagent-page">
            <div class="reagent-hero">
              <div><span>试剂全流程管控</span><h2>检化验试剂管理</h2></div>
              <div class="lab-stat-grid"><div v-for="item in reagentStats" :key="item.label" :class="['lab-stat-card', item.tone]"><b>{{ item.value }}</b><span>{{ item.label }}</span></div></div>
            </div>

            <div class="sampling-toolbar reagent-toolbar">
              <div class="sampling-source-tabs">
                <button type="button" :class="{ active: reagentTab === 'overview' }" @click="reagentTab = 'overview'">试剂管理总览</button>
                <button type="button" :class="{ active: reagentTab === 'inventory' }" @click="reagentTab = 'inventory'">试剂库存台账</button>
                <button type="button" :class="{ active: reagentTab === 'cabinets' }" @click="reagentTab = 'cabinets'">试剂柜环境监测</button>
                <button type="button" :class="{ active: reagentTab === 'records' }" @click="reagentTab = 'records'">过程留痕与审批</button>
              </div>
            </div>

            <section v-if="reagentTab === 'overview'" class="reagent-overview-grid">
              <section class="lab-panel reagent-overview-process">
                <div class="lab-panel-head"><div><h3>流程数量总览</h3><span>采购、入库、领用、归还、盘点</span></div></div>
                <div class="reagent-flow-strip"><button v-for="item in reagentFlowSummary" :key="item.label" :class="['reagent-flow-card', item.tone]" type="button" @click="openReagentFlowModule(item.label)"><b>{{ item.value }}</b><span>{{ item.label }}</span></button></div>
              </section>
              <section class="lab-panel reagent-overview-alerts">
                <div class="lab-panel-head"><div><h3>低库存采购提醒</h3><span>需要优先关注的库存事项</span></div><button type="button" @click="reagentTab = 'inventory'">查看库存</button></div>
                <div class="reagent-overview-alert-list"><div v-for="item in lowStockReagents" :key="`stock-${item.code}`"><b>低库存 · {{ item.name }}</b><span>当前 {{ item.stock }}{{ item.unit }} / 阈值 {{ item.threshold }}{{ item.unit }}</span></div><div v-if="!lowStockReagents.length">暂无低库存提醒</div></div>
              </section>
              <section class="lab-panel reagent-overview-cabinet-alerts">
                <div class="lab-panel-head"><div><h3>柜温预警</h3><span>试剂柜环境不符合存放标准</span></div><button type="button" @click="reagentTab = 'cabinets'">查看试剂柜</button></div>
                <div class="reagent-cabinet-alert-list"><button v-for="cabinet in reagentCabinets.filter((item) => reagentCabinetAlert(item))" :key="`cabinet-${cabinet.code}`" type="button" @click="openReagentCabinetHistory(cabinet)"><b>{{ cabinet.name }}</b><span>{{ reagentCabinetAlert(cabinet) }}</span></button><div v-if="!reagentCabinetAlertCount">当前无柜温预警</div></div>
              </section>
              <section class="lab-panel reagent-overview-approval">
                <div class="lab-panel-head"><div><h3>待审批信息</h3><span>采购和领用审批待办</span></div></div>
                <div class="reagent-overview-approval-list"><div v-for="item in reagentFlowRecords.filter((record) => record.approvalStatus === '审批中')" :key="item.recordNo"><b>{{ item.action }} · {{ item.reagentName }}</b><span>{{ item.approvalStep ?? '待审批' }} / {{ item.recordNo }}</span><button type="button" @click="openReagentApprovalFlow(item.recordNo)">查看</button></div><div v-if="!reagentFlowRecords.some((record) => record.approvalStatus === '审批中')">暂无待审批信息</div></div>
              </section>
            </section>

            <section v-else-if="reagentTab === 'inventory'" class="lab-panel reagent-inventory-module">
                <div class="lab-panel-head"><div><h3>试剂库存台账</h3></div><button type="button" @click="showReagentEntryForm = true"><Plus :size="13" /> 新增试剂台账</button></div>
                <table class="lab-table reagent-table"><thead><tr><th>试剂编号</th><th>试剂名称</th><th>类别</th><th>规格</th><th>库存</th><th>预警阈值</th><th>试剂位置</th><th>保管人</th><th>状态</th></tr></thead><tbody><tr v-for="item in reagentStocks" :key="item.code"><td>{{ item.code }}</td><td>{{ item.name }}</td><td><span :class="['reagent-type', item.type === '易制毒试剂' ? 'danger' : 'normal']">{{ item.type }}</span></td><td>{{ item.spec }}</td><td><b :class="{ 'unqualified-text': item.stock <= item.threshold }">{{ item.stock }}{{ item.unit }}</b></td><td>{{ item.threshold }}{{ item.unit }}</td><td>{{ item.location }}</td><td>{{ item.keeper }}</td><td><span :class="['lab-status', item.stock <= item.threshold || item.type === '易制毒试剂' ? 'warn' : 'ok']">{{ item.status }}</span></td></tr></tbody></table>
            </section>

            <section v-else-if="reagentTab === 'cabinets'" class="lab-panel env-panel environment-module reagent-cabinet-module">
              <div class="reagent-cabinet-type-tabs" role="tablist" aria-label="试剂柜类型">
                <button type="button" role="tab" :aria-selected="activeReagentCabinetType === '普通试剂柜'" :class="{ active: activeReagentCabinetType === '普通试剂柜' }" @click="selectReagentCabinetType('普通试剂柜')"><b>普通试剂柜</b><span>{{ reagentStocks.filter((item) => item.type === '普通试剂').length }} 种试剂</span></button>
                <button type="button" role="tab" :aria-selected="activeReagentCabinetType === '易制毒双人双锁柜'" :class="['controlled', { active: activeReagentCabinetType === '易制毒双人双锁柜' }]" @click="selectReagentCabinetType('易制毒双人双锁柜')"><b>易制毒试剂柜</b><span>{{ reagentStocks.filter((item) => item.type === '易制毒试剂').length }} 种试剂</span></button>
                <div class="reagent-cabinet-inline-actions"><button type="button" class="cabinet-add-button" title="新增试剂柜信息" @click="openReagentCabinetForm"><Plus :size="15" /><span>新增试剂柜</span></button><button type="button" class="cabinet-stock-add-button" title="添加柜内试剂" @click="openReagentEntryForCabinet"><Plus :size="15" /><span>添加柜内试剂</span></button></div>
              </div>
              <section class="environment-live-section reagent-cabinet-live-section">
                <div class="environment-live-head"><div><i></i><b>{{ activeReagentCabinetType === '普通试剂柜' ? '普通试剂柜' : '易制毒试剂柜' }}实时温湿度</b></div><div class="environment-live-summary"><span>柜体数量 <b>{{ visibleReagentCabinets.length }}</b></span><span>温湿度标准 <b>15-25℃ / 30-60%</b></span></div></div>
                <div class="environment-live-grid reagent-cabinet-live-grid">
                  <div v-for="item in visibleReagentCabinetRecords" :key="item.cabinetCode" role="button" tabindex="0" :class="['environment-live-item', 'reagent-cabinet-live-item', { alert: item.status !== '正常', selected: selectedReagentCabinetCode === item.cabinetCode }]" @click="selectReagentCabinet(reagentCabinetByCode(item.cabinetCode))" @keydown.enter="selectReagentCabinet(reagentCabinetByCode(item.cabinetCode))"><div class="reagent-cabinet-live-title"><span><b>{{ item.cabinetName }}</b><small>{{ item.cabinetCode }} · {{ item.time }}</small></span><button type="button" title="查看历史温湿度趋势" aria-label="查看历史温湿度趋势" @click.stop="openReagentCabinetTrend(reagentCabinetByCode(item.cabinetCode))"><ChartNoAxesCombined :size="15" /></button></div><strong><small>温度</small>{{ item.temperature }}℃</strong><strong><small>湿度</small>{{ item.humidity }}%</strong><em :class="['lab-status', environmentStatusTone(item.status)]">{{ item.status }}</em></div>
                </div>
              </section>
              <section class="reagent-cabinet-inventory-section">
                <div class="environment-detail-head"><div><h4>{{ selectedReagentCabinet?.name }} · 柜内试剂明细</h4><span>当前 {{ selectedReagentCabinet?.temperature }}℃ / {{ selectedReagentCabinet?.humidity }}%RH · 存放标准 {{ selectedReagentCabinet ? reagentCabinetStorageStandard(selectedReagentCabinet).temperature : '15-25℃' }} / {{ selectedReagentCabinet ? reagentCabinetStorageStandard(selectedReagentCabinet).humidity : '30-60%' }}</span></div><b>{{ selectedReagentCabinetItems.length }} 种</b></div>
                <div class="reagent-cabinet-stock-filters"><label><Search :size="14" /><input v-model="reagentCabinetStockKeyword" placeholder="搜索试剂编号、名称、规格或保管人" /></label><label>状态<select v-model="reagentCabinetStockStatusFilter"><option value="all">全部试剂</option><option value="normal">试剂环境正常</option><option value="abnormal">试剂环境异常</option></select></label><span>显示 {{ filteredSelectedReagentCabinetItems.length }} / {{ selectedReagentCabinetItems.length }} 种</span></div>
                <div class="environment-detail-table-wrap reagent-cabinet-inventory-table-wrap"><table class="lab-table reagent-cabinet-inventory-table"><thead><tr><th>试剂编号</th><th>试剂名称</th><th>规格</th><th>当前库存</th><th>预警阈值</th><th>保管人</th><th>存放温度</th><th>存放湿度</th><th>状态</th></tr></thead><tbody><tr v-for="item in filteredSelectedReagentCabinetItems" :key="item.code"><td>{{ item.code }}</td><td><b>{{ item.name }}</b><span :class="['reagent-type', item.type === '易制毒试剂' ? 'danger' : 'normal']">{{ item.type }}</span></td><td>{{ item.spec }}</td><td><b :class="{ 'unqualified-text': item.stock <= item.threshold }">{{ item.stock }}{{ item.unit }}</b></td><td>{{ item.threshold }}{{ item.unit }}</td><td>{{ item.keeper }}</td><td>{{ selectedReagentCabinet ? reagentCabinetStorageStandard(selectedReagentCabinet).temperature : '—' }}</td><td>{{ selectedReagentCabinet ? reagentCabinetStorageStandard(selectedReagentCabinet).humidity : '—' }}</td><td><span :class="['lab-status', selectedReagentCabinet && reagentCabinetAlert(selectedReagentCabinet) ? 'danger' : 'ok']">{{ reagentEnvironmentStatus(selectedReagentCabinet) }}</span></td></tr><tr v-if="!filteredSelectedReagentCabinetItems.length"><td colspan="9">当前筛选条件下暂无试剂。</td></tr></tbody></table></div>
              </section>
            </section>

            <div v-if="reagentTab === 'records'" class="reagent-process-page">
              <div class="reagent-process-tabs">
                <button v-for="tab in reagentProcessTabs" :key="tab" type="button" :class="{ active: reagentProcessTab === tab }" @click="reagentProcessTab = tab">{{ tab }}</button>
              </div>

              <section v-if="reagentProcessTab === '采购'" class="lab-panel">
                <div class="lab-panel-head"><div><h3>采购申请</h3><span>新增采购信息后进入编制人、审核人、批准人审批</span></div><button type="button" @click="openReagentFlowForm('采购')"><Plus :size="13" /> 新增采购信息</button></div>
                <div class="reagent-process-table-wrap"><table class="lab-table reagent-flow-table"><thead><tr><th>采购单号</th><th>试剂名称</th><th>数量</th><th>供应商</th><th>预计入库位置</th><th>申请时间</th><th>审批状态</th><th>操作</th></tr></thead><tbody><tr v-for="item in reagentFlowRecords.filter((record) => record.action === '采购')" :key="item.recordNo"><td>{{ item.recordNo }}</td><td>{{ item.reagentName }}</td><td>{{ item.quantity }}{{ item.unit }}</td><td>{{ item.supplier }}</td><td>{{ item.location }}</td><td>{{ item.time }}</td><td><span :class="['lab-status', item.approvalStatus === '审批通过' ? 'ok' : 'warn']">{{ reagentProcessStatusText(item) }}</span></td><td><button type="button" class="reagent-flow-link" @click="openReagentApprovalFlow(item.recordNo)">审批流程</button><button v-if="item.approvalStatus === '审批通过' && !reagentInboundRecordForProcurement(item)" type="button" class="reagent-flow-link" @click="openInboundFromProcurement(item)">生成入库单</button><button v-else-if="item.approvalStatus === '审批通过' && reagentInboundRecordForProcurement(item)" type="button" class="reagent-flow-link" @click="openReagentInboundReport(reagentInboundRecordForProcurement(item)!)"><Eye :size="13" /> 查看入库单</button></td></tr><tr v-if="!reagentFlowRecords.some((record) => record.action === '采购')"><td colspan="8">暂无采购申请，请新增采购信息。</td></tr></tbody></table></div>
              </section>

              <section v-else-if="reagentProcessTab === '入库'" class="lab-panel">
                <div class="lab-panel-head"><div><h3>入库登记</h3></div><button type="button" @click="openReagentFlowForm('入库')"><Plus :size="13" /> 新增入库</button></div>
                <div class="reagent-process-table-wrap"><table class="lab-table reagent-flow-table"><thead><tr><th>入库时间</th><th>入库单号</th><th>来源采购单</th><th>试剂名称</th><th>数量</th><th>试剂位置</th><th>经办人</th><th>状态</th><th>操作</th></tr></thead><tbody><tr v-for="item in reagentInboundRecords" :key="item.recordNo"><td>{{ item.time }}</td><td>{{ item.recordNo }}</td><td>{{ item.sourceRecordNo ?? '直接入库' }}</td><td>{{ item.reagentName }}</td><td>{{ item.quantity }}{{ item.unit }}</td><td>{{ item.location }}</td><td>{{ item.handler }}</td><td><span class="lab-status ok">已入库</span></td><td><button type="button" class="reagent-flow-link" @click="openReagentInboundReport(item)"><Eye :size="13" /> 查看入库单</button></td></tr></tbody></table></div>
              </section>

              <section v-else-if="reagentProcessTab === '领用'" class="lab-panel">
                <div class="lab-panel-head"><div><h3>领用申请与审批</h3><span>领用提交后进入审批，审批完成才转为领用中</span></div><button type="button" @click="openReagentFlowForm('领用')"><Plus :size="13" /> 新增领用</button></div>
                <div class="reagent-process-table-wrap"><table class="lab-table reagent-flow-table"><thead><tr><th>申请时间</th><th>领用单号</th><th>试剂名称</th><th>领用数量</th><th>试剂位置</th><th>领用人</th><th>状态</th><th>操作</th></tr></thead><tbody><tr v-for="item in reagentUsageRecords" :key="item.recordNo"><td>{{ item.time }}</td><td>{{ item.recordNo }}</td><td>{{ item.reagentName }}</td><td>{{ item.quantity }}{{ item.unit }}</td><td>{{ item.location }}</td><td>{{ item.handler }}</td><td><span :class="['lab-status', item.processStatus === '领用中' ? 'info' : 'warn']">{{ reagentProcessStatusText(item) }}</span></td><td><button type="button" class="reagent-flow-link" @click="openReagentApprovalFlow(item.recordNo)">审批流程</button></td></tr></tbody></table></div>
              </section>

              <section v-else-if="reagentProcessTab === '归还'" class="lab-panel">
                <div class="lab-panel-head"><div><h3>归还登记</h3><span>审批完成的领用单自动进入此处，归还时仅登记归还信息</span></div></div>
                <div class="reagent-process-table-wrap"><table class="lab-table reagent-flow-table"><thead><tr><th>领用单号</th><th>领用时间</th><th>试剂名称</th><th>领用数量</th><th>试剂位置</th><th>领用人</th><th>归还状态</th><th>操作</th></tr></thead><tbody><tr v-for="item in reagentReturnWorkflowRecords" :key="item.recordNo"><td>{{ item.recordNo }}</td><td>{{ item.time }}</td><td>{{ item.reagentName }}</td><td>{{ item.quantity }}{{ item.unit }}</td><td>{{ item.location }}</td><td>{{ item.handler }}</td><td><span :class="['lab-status', item.processStatus === '领用中' ? 'warn' : 'ok']">{{ item.processStatus === '领用中' ? '待归还' : '已归还' }}</span></td><td><button v-if="item.processStatus === '领用中'" type="button" class="reagent-flow-link" @click="openReagentFlowForm('归还', item.recordNo)">归还</button><span v-else class="maintenance-empty">已完成</span></td></tr></tbody></table></div>
              </section>

              <section v-else class="lab-panel">
                <div class="lab-panel-head"><div><h3>归还后盘点</h3><span>归还提交后自动生成盘点记录，无需重复录入试剂信息</span></div></div>
                <div class="reagent-process-table-wrap"><table class="lab-table reagent-flow-table"><thead><tr><th>盘点单号</th><th>关联归还单</th><th>生成时间</th><th>试剂名称</th><th>归还数量</th><th>试剂位置</th><th>盘点状态</th><th>盘点结果</th><th>操作</th></tr></thead><tbody><tr v-for="item in reagentInventoryRecords" :key="item.recordNo"><td>{{ item.recordNo }}</td><td>{{ item.sourceRecordNo }}</td><td>{{ item.time }}</td><td>{{ item.reagentName }}</td><td>{{ item.quantity }}{{ item.unit }}</td><td>{{ item.location }}</td><td><span :class="['lab-status', item.processStatus === '已盘点' ? 'ok' : 'warn']">{{ item.processStatus ?? '待盘点' }}</span></td><td>{{ item.inventoryResult ?? '待确认' }}<small v-if="item.inventoryActualQuantity !== undefined"> / 实盘 {{ item.inventoryActualQuantity }}{{ item.unit }}</small></td><td><button v-if="item.processStatus !== '已盘点'" type="button" class="reagent-flow-link" @click="openReagentInventoryConfirm(item)"><CheckCircle2 :size="13" /> 确认盘点</button><button v-else type="button" class="reagent-flow-link" @click="openReagentInventoryConfirm(item, true)"><Eye :size="13" /> 查看盘点单</button></td></tr></tbody></table></div>
              </section>
            </div>
          </div>

          <div v-if="showReagentEntryForm" class="process-dialog-mask" @click.self="showReagentEntryForm = false">
            <section class="send-sample-dialog reagent-entry-dialog">
              <div class="process-dialog-head"><div><h2>新增试剂台账</h2><span>登记试剂基础信息、库存、位置与保管人</span></div><button type="button" @click="showReagentEntryForm = false"><X :size="12" /> 关闭</button></div>
              <div class="send-form">
                <label>试剂编号<input v-model="reagentEntryForm.reagentCode" placeholder="如：SJ-PT-001" /></label>
                <label>试剂名称<input v-model="reagentEntryForm.name" placeholder="如：无水乙醇" /></label>
                <label>规格<input v-model="reagentEntryForm.spec" placeholder="如：500ml/瓶" /></label>
                <label>单位<select v-model="reagentEntryForm.unit"><option>瓶</option><option>L</option><option>kg</option><option>g</option></select></label>
                <label>数量<input v-model.number="reagentEntryForm.quantity" type="number" min="0" /></label>
                <label>预警阈值<input v-model.number="reagentEntryForm.threshold" type="number" min="0" /></label>
                <label>试剂位置<input v-model="reagentEntryForm.location" placeholder="如：试剂柜A01" /></label>
                <label>保管人<input v-model="reagentEntryForm.keeper" placeholder="如：质检员A" /></label>
                <label>类别<select v-model="reagentEntryForm.type"><option>普通试剂</option><option>易制毒试剂</option></select></label>
                <button type="button" @click="addReagentLedger">确认新增</button>
              </div>
            </section>
          </div>

          <div v-if="showReagentInboundReport && selectedReagentInboundRecord" class="process-dialog-mask" @click.self="showReagentInboundReport = false">
            <section class="reagent-inbound-report-dialog">
              <div class="process-dialog-head"><div><h2>试剂入库单</h2><span>{{ selectedReagentInboundRecord.recordNo }} · {{ selectedReagentInboundRecord.sourceRecordNo ? `来源采购单 ${selectedReagentInboundRecord.sourceRecordNo}` : '直接入库' }}</span></div><button type="button" @click="showReagentInboundReport = false"><X :size="13" /> 关闭</button></div>
              <article class="reagent-inbound-report-body">
                <header><div><span>检化验试剂管理</span><h1>试剂入库单</h1></div><div class="reagent-inbound-report-code"><b>{{ selectedReagentInboundRecord.recordNo }}</b><small>{{ selectedReagentInboundRecord.time }}</small></div></header>
                <div class="reagent-inbound-report-meta"><div><span>试剂名称</span><b>{{ selectedReagentInboundRecord.reagentName }}</b></div><div><span>试剂编号</span><b>{{ selectedReagentInboundRecord.reagentCode }}</b></div><div><span>入库数量</span><b>{{ selectedReagentInboundRecord.quantity }}{{ selectedReagentInboundRecord.unit }}</b></div><div><span>入库位置</span><b>{{ selectedReagentInboundRecord.location }}</b></div><div><span>经办人</span><b>{{ selectedReagentInboundRecord.handler }}</b></div><div><span>入库状态</span><b class="reagent-report-ok">已入库</b></div></div>
                <section><h3>入库信息</h3><table><tbody><tr><th>来源采购单</th><td>{{ selectedReagentInboundRecord.sourceRecordNo ?? '直接入库' }}</td><th>规格</th><td>{{ reagentStocks.find((item) => item.code === selectedReagentInboundRecord.reagentCode)?.spec ?? '—' }}</td></tr><tr><th>业务说明</th><td colspan="3">{{ selectedReagentInboundRecord.remark || '无' }}</td></tr></tbody></table></section>
                <footer><span>编制人：{{ selectedReagentInboundRecord.handler || '系统' }}</span><span>入库确认：{{ selectedReagentInboundRecord.time }}</span></footer>
              </article>
            </section>
          </div>

          <div v-if="showReagentInventoryConfirmDialog && selectedReagentInventoryRecord" class="process-dialog-mask" @click.self="showReagentInventoryConfirmDialog = false">
            <section class="reagent-inventory-confirm-dialog">
              <div class="process-dialog-head"><div><h2>{{ reagentInventoryConfirmReadOnly ? '盘点单详情' : '确认盘点信息' }}</h2><span>{{ selectedReagentInventoryRecord.recordNo }} · {{ reagentInventoryConfirmReadOnly ? '已确认盘点记录' : '请核对账面数量与实盘数量' }}</span></div><button type="button" @click="showReagentInventoryConfirmDialog = false; reagentInventoryConfirmReadOnly = false"><X :size="13" /> 关闭</button></div>
              <div class="reagent-inventory-confirm-body"><div class="reagent-inventory-confirm-summary"><div><span>试剂名称</span><b>{{ selectedReagentInventoryRecord.reagentName }}</b></div><div><span>存放位置</span><b>{{ selectedReagentInventoryRecord.location }}</b></div><div><span>账面数量</span><b>{{ selectedReagentInventoryRecord.quantity }}{{ selectedReagentInventoryRecord.unit }}</b></div><div><span>关联归还单</span><b>{{ selectedReagentInventoryRecord.sourceRecordNo ?? '—' }}</b></div></div><div class="reagent-inventory-confirm-form"><label>实盘数量<input v-model.number="reagentInventoryConfirmForm.actualQuantity" :readonly="reagentInventoryConfirmReadOnly" type="number" min="0" step="0.1" /></label><label>盘点结果<input :value="selectedReagentInventoryRecord.inventoryResult ?? (Number(reagentInventoryConfirmForm.actualQuantity) === selectedReagentInventoryRecord.quantity ? '账实相符' : '存在差异')" readonly /></label><label>确认人<input v-model="reagentInventoryConfirmForm.confirmer" :readonly="reagentInventoryConfirmReadOnly" placeholder="请输入确认人" /></label><label class="full">盘点备注<textarea v-model="reagentInventoryConfirmForm.remark" :readonly="reagentInventoryConfirmReadOnly" placeholder="可填写封签、包装、数量差异等盘点说明"></textarea></label></div><button v-if="!reagentInventoryConfirmReadOnly" type="button" class="reagent-inventory-confirm-button" @click="confirmReagentInventory"><CheckCircle2 :size="15" /> 确认盘点并完成</button><div v-else class="reagent-inventory-confirmed-tip"><CheckCircle2 :size="15" /> 已由 {{ selectedReagentInventoryRecord.inventoryConfirmedBy || '系统' }} 于 {{ selectedReagentInventoryRecord.inventoryConfirmedAt || selectedReagentInventoryRecord.time }} 确认</div></div>
            </section>
          </div>

          <div v-if="showReagentCabinetForm" class="process-dialog-mask" @click.self="showReagentCabinetForm = false">
            <section class="send-sample-dialog reagent-cabinet-form-dialog">
              <div class="process-dialog-head"><div><h2>新增试剂柜信息</h2><span>登记柜体类型、编号及当前环境参数</span></div><button type="button" @click="showReagentCabinetForm = false"><X :size="12" /> 关闭</button></div>
              <div class="send-form">
                <label>试剂柜类型<select v-model="reagentCabinetForm.type"><option value="普通试剂柜">普通试剂柜</option><option value="易制毒双人双锁柜">易制毒双人双锁柜</option></select></label>
                <label>试剂柜编号<input v-model="reagentCabinetForm.code" placeholder="留空自动生成，如 A04" /></label>
                <label class="full">试剂柜名称<input v-model="reagentCabinetForm.name" placeholder="留空按类型和编号自动生成" /></label>
                <label>当前温度（℃）<input v-model.number="reagentCabinetForm.temperature" type="number" min="-20" max="60" step="0.1" /></label>
                <label>当前湿度（%RH）<input v-model.number="reagentCabinetForm.humidity" type="number" min="0" max="100" /></label>
                <button type="button" @click="addReagentCabinet"><Plus :size="14" /> 确认新增试剂柜</button>
              </div>
            </section>
          </div>

          <div v-if="showReagentFlowForm" class="process-dialog-mask" @click.self="showReagentFlowForm = false">
            <section class="send-sample-dialog reagent-flow-dialog">
              <div class="process-dialog-head"><div><h2>{{ reagentFlowForm.action }}{{ reagentFlowForm.action === '领用' ? '申请' : '登记' }}</h2><span>{{ reagentFlowForm.action === '领用' ? '提交后按检验报告流程逐级审批' : reagentFlowForm.action === '归还' ? '自动带入领用信息，仅填写归还人、时间与归还数量' : '登记到货试剂及入库信息' }}</span></div><button type="button" @click="showReagentFlowForm = false"><X :size="12" /> 关闭</button></div>
              <div class="reagent-flow-form-body">
                <template v-if="reagentFlowForm.action === '采购'">
                  <label>试剂编号<input v-model="reagentFlowForm.reagentCode" placeholder="如：SJ-PT-004" /></label>
                  <label>试剂名称<input v-model="reagentFlowForm.name" placeholder="请输入采购试剂名称" /></label>
                  <label>类别<select v-model="reagentFlowForm.type"><option>普通试剂</option><option>易制毒试剂</option></select></label>
                  <label>规格<input v-model="reagentFlowForm.spec" placeholder="如：500ml/瓶" /></label>
                  <label>采购数量<input v-model.number="reagentFlowForm.quantity" type="number" min="0" /></label>
                  <label>单位<select v-model="reagentFlowForm.unit"><option>瓶</option><option>L</option><option>kg</option><option>g</option></select></label>
                  <label>供应商<input v-model="reagentFlowForm.supplier" placeholder="请输入供应商" /></label>
                  <label>预计入库位置<input v-model="reagentFlowForm.location" placeholder="如：普通试剂柜A01" /></label>
                </template>

                <template v-else-if="reagentFlowForm.action === '入库'">
                  <label>试剂编号<input v-model="reagentFlowForm.reagentCode" placeholder="如：SJ-PT-004" /></label>
                  <label>试剂名称<input v-model="reagentFlowForm.name" placeholder="请输入试剂名称" /></label>
                  <label>类别<select v-model="reagentFlowForm.type"><option>普通试剂</option><option>易制毒试剂</option></select></label>
                  <label>规格<input v-model="reagentFlowForm.spec" placeholder="如：500ml/瓶" /></label>
                  <label>入库数量<input v-model.number="reagentFlowForm.quantity" type="number" min="0" /></label>
                  <label>单位<select v-model="reagentFlowForm.unit"><option>瓶</option><option>L</option><option>kg</option><option>g</option></select></label>
                  <label>预警阈值<input v-model.number="reagentFlowForm.threshold" type="number" min="0" /></label>
                  <label>试剂位置<input v-model="reagentFlowForm.location" placeholder="如：普通试剂柜A01" /></label>
                  <label>保管人<input v-model="reagentFlowForm.keeper" placeholder="请输入保管人" /></label>
                </template>

                <template v-else-if="reagentFlowForm.action === '领用'">
                  <label>领用试剂<select v-model="reagentFlowForm.reagentCode" @change="applyReagentStockToFlowForm"><option value="">请选择试剂</option><option v-for="item in reagentStocks" :key="item.code" :value="item.code">{{ item.name }}（{{ item.code }}）</option></select></label>
                  <label>试剂位置<input :value="reagentFlowForm.location" readonly /></label>
                  <label>可用库存<input :value="reagentStocks.find((item) => item.code === reagentFlowForm.reagentCode)?.stock ?? 0" readonly /></label>
                  <label>领用数量<input v-model.number="reagentFlowForm.quantity" type="number" min="0" /></label>
                </template>

                <template v-else>
                  <label>关联领用单<input :value="reagentFlowSourceRecordNo" readonly /></label>
                  <label>试剂名称<input :value="reagentFlowForm.name" readonly /></label>
                  <label>试剂位置<input :value="reagentFlowForm.location" readonly /></label>
                  <label>归还数量<input v-model.number="reagentFlowForm.quantity" type="number" min="1" step="1" /></label>
                </template>

                <label>{{ reagentFlowForm.action === '归还' ? '归还人' : reagentFlowForm.action === '采购' ? '申请人' : '处理人' }}<input v-model="reagentFlowForm.handler" :placeholder="reagentFlowForm.action === '归还' ? '请输入归还人' : reagentFlowForm.action === '采购' ? '请输入采购申请人' : '请输入处理人'" /></label>
                <label>{{ reagentFlowForm.action === '归还' ? '归还时间' : reagentFlowForm.action === '采购' ? '申请时间' : '操作时间' }}<input v-model="reagentFlowForm.time" type="datetime-local" /></label>
                <label v-if="reagentFlowForm.action !== '归还'" class="full">业务说明<textarea v-model="reagentFlowForm.remark" placeholder="请输入本次业务说明"></textarea></label>
                <button type="button" @click="saveReagentFlow">{{ reagentFlowForm.action === '领用' || reagentFlowForm.action === '采购' ? '提交审批' : `确认${reagentFlowForm.action}` }}</button>
              </div>
            </section>
          </div>

          <div v-if="showReagentCabinetHistory" class="process-dialog-mask" @click.self="showReagentCabinetHistory = false">
            <section class="reagent-cabinet-dialog">
              <div class="process-dialog-head"><div><h2>{{ selectedReagentCabinet?.name }} 详情</h2><span>{{ selectedReagentCabinet?.type }} · 存放标准与柜内试剂</span></div><button type="button" @click="showReagentCabinetHistory = false"><X :size="12" /> 关闭</button></div>
              <div class="reagent-cabinet-history-body">
                <div class="cabinet-history-summary"><div><span>当前温度</span><b>{{ selectedReagentCabinet?.temperature }}℃</b></div><div><span>当前湿度</span><b>{{ selectedReagentCabinet?.humidity }}%</b></div><div><span>最近采集</span><b>{{ selectedReagentCabinet?.sampledAt }}</b></div></div>
                <div v-if="selectedReagentCabinet" :class="['reagent-cabinet-standard-banner', { alert: Boolean(reagentCabinetAlert(selectedReagentCabinet)) }]"><div><b>{{ reagentCabinetAlert(selectedReagentCabinet) || '当前柜体环境符合存放标准' }}</b><span>温度标准 {{ reagentCabinetStorageStandard(selectedReagentCabinet).temperature }} · 湿度标准 {{ reagentCabinetStorageStandard(selectedReagentCabinet).humidity }}</span></div></div>
                <section class="reagent-cabinet-stock-section"><div class="section-subhead"><div><h4>柜内试剂</h4><span>库存台账与存放标准</span></div><span>{{ selectedReagentCabinetItems.length }} 种</span></div><div class="reagent-cabinet-stock-table-wrap"><table class="lab-table"><thead><tr><th>试剂编号</th><th>试剂名称</th><th>规格</th><th>库存</th><th>存放温度</th><th>存放湿度</th><th>当前状态</th></tr></thead><tbody><tr v-for="item in selectedReagentCabinetItems" :key="item.code"><td>{{ item.code }}</td><td>{{ item.name }}</td><td>{{ item.spec }}</td><td>{{ item.stock }}{{ item.unit }}</td><td>{{ selectedReagentCabinet ? reagentCabinetStorageStandard(selectedReagentCabinet).temperature : '—' }}</td><td>{{ selectedReagentCabinet ? reagentCabinetStorageStandard(selectedReagentCabinet).humidity : '—' }}</td><td><span :class="['lab-status', selectedReagentCabinet && reagentCabinetAlert(selectedReagentCabinet) ? 'danger' : 'ok']">{{ reagentEnvironmentStatus(selectedReagentCabinet) }}</span></td></tr><tr v-if="!selectedReagentCabinetItems.length"><td colspan="7">该试剂柜暂无关联试剂。</td></tr></tbody></table></div></section>
                <div class="cabinet-history-table-wrap"><table class="lab-table"><thead><tr><th>采集时间</th><th>柜内温度</th><th>柜内湿度</th></tr></thead><tbody><tr v-for="item in reagentCabinetHistoryRows" :key="item.time"><td>{{ item.time }}</td><td>{{ item.temperature }}℃</td><td>{{ item.humidity }}%</td></tr></tbody></table></div>
              </div>
            </section>
          </div>

          <div v-if="showReagentCabinetTrendDialog" class="process-dialog-mask" @click.self="showReagentCabinetTrendDialog = false">
            <section class="environment-trend-dialog">
              <div class="process-dialog-head"><div><h2>试剂柜温湿度趋势</h2><span>{{ activeReagentCabinetRangeLabel }} · {{ activeReagentCabinetFilter === '全部试剂柜' ? '全部试剂柜' : reagentCabinetByCode(activeReagentCabinetFilter).name }}</span></div><button type="button" @click="showReagentCabinetTrendDialog = false"><X :size="14" /> 关闭</button></div>
              <div class="environment-trend-dialog-body">
                <div class="environment-trend-toolbar"><div class="env-range-tabs"><button v-for="range in reagentCabinetTrendRanges" :key="range.key" type="button" :class="{ active: activeReagentCabinetRange === range.key }" @click="activeReagentCabinetRange = range.key">{{ range.label }}</button></div><label>当前柜体 <b>{{ reagentCabinetByCode(activeReagentCabinetFilter).name }}</b></label></div>
                <div class="environment-trend-summary"><span>统计维度 <b>{{ activeReagentCabinetRangeLabel }}</b></span><span>平均温度 <b>{{ reagentCabinetTrendAvgTemperature }}</b></span><span>平均湿度 <b>{{ reagentCabinetTrendAvgHumidity }}</b></span></div>
                <div class="env-trend-chart environment-popup-chart">
                  <div class="chart-legend"><span class="temp">温度</span><span class="humidity">湿度</span></div>
                  <div v-if="hoveredReagentCabinetPoint" class="chart-tooltip">{{ hoveredReagentCabinetPoint.label }} · {{ formatEnvironmentTooltipDate(activeReagentCabinetRange, hoveredReagentCabinetPoint.date) }} · 温度 {{ hoveredReagentCabinetPoint.temperature }}℃ · 湿度 {{ hoveredReagentCabinetPoint.humidity }}%</div>
                  <svg :viewBox="`0 0 ${environmentChartWidth} ${environmentChartHeight}`" role="img" aria-label="试剂柜温湿度趋势图">
                    <line :x1="environmentChartPaddingX" :y1="environmentChartPaddingY" :x2="environmentChartPaddingX" :y2="environmentChartLabelY - 30" />
                    <line :x1="environmentChartPaddingX" :y1="environmentChartLabelY - 30" :x2="environmentChartWidth - environmentChartPaddingX" :y2="environmentChartLabelY - 30" />
                    <template v-for="(point, index) in reagentCabinetChartRows" :key="`${point.date}-${point.label}-axis`"><line class="chart-x-guide" :x1="point.x" :y1="environmentChartPaddingY" :x2="point.x" :y2="environmentChartLabelY - 30" /><text v-if="showReagentCabinetAxisLabel(index, reagentCabinetChartRows.length)" class="chart-axis-text" :x="point.x" :y="environmentChartLabelY" text-anchor="middle">{{ point.label }}</text></template>
                    <polyline class="temperature-line" :points="reagentCabinetTemperaturePoints" /><polyline class="humidity-line" :points="reagentCabinetHumidityPoints" />
                    <template v-for="point in reagentCabinetChartRows" :key="`${point.date}-${point.label}`"><circle class="temperature-dot" r="4" v-bind="environmentPointPosition(point, 'temperature')" @mouseenter="hoveredReagentCabinetPoint = point" @mouseleave="hoveredReagentCabinetPoint = null" /><circle class="humidity-dot" r="4" v-bind="environmentPointPosition(point, 'humidity')" @mouseenter="hoveredReagentCabinetPoint = point" @mouseleave="hoveredReagentCabinetPoint = null" /></template>
                  </svg>
                </div>
              </div>
            </section>
          </div>

          <div v-if="showReagentApprovalFlow" class="process-dialog-mask" @click.self="showReagentApprovalFlow = false">
            <section class="approval-flow-dialog">
              <div class="process-dialog-head"><div><h2>试剂流程审批</h2><span>{{ selectedReagentFlowRecord?.recordNo ?? '' }}</span></div><button type="button" @click="showReagentApprovalFlow = false">关闭</button></div>
              <div class="approval-flow-body">
                <div class="approval-flow-summary">
                  <div><span>试剂名称</span><b>{{ selectedReagentFlowRecord?.reagentName }}</b></div>
                  <div><span>当前环节</span><b>{{ selectedReagentFlowRecord?.approvalStep ?? selectedReagentFlowRecord?.approvalStatus ?? '未提交' }}</b></div>
                  <div><span>过程状态</span><b>{{ selectedReagentFlowRecord?.approvalStatus ?? '未提交' }}</b></div>
                </div>
                <div class="approval-route">
                  <template v-for="(item, idx) in reagentApprovalFlowItems(selectedReagentFlowRecord)" :key="item.name">
                    <div v-if="idx > 0" :class="['approval-route-arrow', item.state === 'done' ? 'done' : '']"></div>
                    <div :class="['approval-route-node', item.state]">
                      <span><component :is="item.icon" :size="18" stroke-width="2.2" /></span>
                      <b>{{ item.name }}</b>
                      <small>{{ item.desc }}</small>
                      <em>{{ item.state === 'active' ? '当前环节' : item.state === 'done' ? '已完成' : '未开始' }}</em>
                    </div>
                  </template>
                </div>
                <section class="approval-history-card">
                  <div class="history-mode"><span>审批历史</span><label><input type="radio" checked /> 正序</label><label><input type="radio" /> 倒序</label></div>
                  <div class="approval-timeline">
                    <div v-for="item in [...(selectedReagentFlowRecord?.approvalHistory ?? [])].reverse()" :key="`${item.title}-${item.time}`" class="approval-timeline-item">
                      <i></i><b>{{ item.title }}</b><span>{{ item.user }} {{ item.time }}</span>
                    </div>
                  </div>
                  <div v-if="selectedReagentFlowRecord && canApproveReagentRecord(selectedReagentFlowRecord)" class="approval-actions inline">
                    <div>当前节点：{{ selectedReagentFlowRecord.approvalStep }}，当前用户：{{ currentLoginUser?.name }}（{{ currentLoginUser?.role }}）</div>
                    <button type="button" @click="approveReagentRecord(selectedReagentFlowRecord.recordNo, 'pass')"><CheckCircle2 :size="13" /> 审批通过/同意</button>
                    <button v-if="currentLoginUser?.role !== '编制人'" type="button" @click="approveReagentRecord(selectedReagentFlowRecord.recordNo, 'return')"><CornerDownLeft :size="13" /> 不同意退回修改</button>
                  </div>
                  <div v-else class="approval-tip">当前登录用户无本节点审批权限，仅可查看流转与历史意见。</div>
                </section>
              </div>
            </section>
          </div>
        </section>

        <section v-else-if="activeMenu === 'system'" class="inspection-page">
          <div class="system-user-page">
            <div class="sampling-toolbar">
              <div class="module-title-block">
                <h2>用户管理</h2>
                <span>用于演示登录用户、角色权限与质检报告逐级审批绑定</span>
              </div>
              <button class="primary-action" type="button" @click="openUserForm('create')"><Plus :size="14" /> 新增用户</button>
            </div>

            <div class="sampling-table-card">
              <table class="sampling-table">
                <thead>
                  <tr>
                    <th>登录账号</th>
                    <th>用户姓名</th>
                    <th>审批角色</th>
                    <th>所属部门</th>
                    <th>联系电话</th>
                    <th>状态</th>
                    <th>说明</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="user in systemUsers" :key="user.username">
                    <td>{{ user.username }}</td>
                    <td>{{ user.name }}</td>
                    <td><span class="sampling-status">{{ user.role }}</span></td>
                    <td>{{ user.department }}</td>
                    <td>{{ user.phone }}</td>
                    <td>{{ user.status }}</td>
                    <td>密码：{{ user.password }}</td>
                    <td><div class="sampling-actions"><button type="button" @click="openUserForm('edit', user)"><Edit3 :size="13" /> 编辑</button><button type="button" @click="toggleUserStatus(user.username)">{{ user.status === '启用' ? '停用' : '启用' }}</button><button type="button" @click="deleteUser(user.username)"><Trash2 :size="13" /> 删除</button></div></td>
                  </tr>
                </tbody>
              </table>
            </div>

            <div class="user-flow-card">
              <h3>审批流转说明</h3>
              <p>质检结果提交后依次进入审核人、负责人、批准人审批；全部审批通过后生成正式质检报告。</p>
              <p>当前节点审批人可以填写审批意见并选择同意或退回修改，意见会保存在历史审批意见中。当前登录用户由登录页账号决定，不允许在审批页面手动切换角色。</p>
            </div>
          </div>
        </section>

        <div v-if="showSamplingForm" class="process-dialog-mask" @click.self="showSamplingForm = false">
          <section class="sampling-form-dialog">
            <div class="process-dialog-head">
              <div><h2>{{ samplingFormMode === 'create' ? (samplingSource === 'internal' ? '内部扦样单新增录入' : '外部扦样单新增录入') : '扦样单编辑' }}</h2><span>按检验事由动态组织录入字段</span></div>
              <button type="button" @click="closeSamplingForm"><X :size="14" /> 关闭</button>
            </div>
            <div class="sampling-form-body">
              <div class="form-line full">
                <label>检验事由</label>
                <select v-model="samplingReason">
                  <option v-for="reason in (samplingSource === 'internal' ? internalReasons : externalReasons)" :key="reason">{{ reason }}</option>
                </select>
              </div>
              <div class="form-grid">
                <label>扦样单编号<input v-model="samplingForm.orderNo" /></label>
                <label>样品编号<input v-model="samplingForm.sampleNo" /></label>
                <label>直属企业<input v-model="samplingForm.company" /></label>
                <label>实际存储库点<input v-model="samplingForm.depot" /></label>
                <label>样品品种<select v-model="samplingForm.grainType"><option v-for="item in grainOptions" :key="item">{{ item }}</option></select></label>
                <label>仓（罐）号<input :value="cargoPositionWarehouseNo(samplingForm.cargoPosition)" readonly /></label>
                <label>货位<select v-model="samplingForm.cargoPosition"><option v-for="item in cargoPositionOptions" :key="item.label" :value="item.label">{{ item.label }}</option></select></label>
                <label v-if="samplingReason.includes('春季') || samplingReason.includes('秋季')">普查编号<input v-model="samplingForm.surveyNo" placeholder="如：CJP-20260704-49" /></label>
                <label>样品名称<input v-model="samplingForm.sampleName" /></label>
                <label>代表样品数量<input v-model="samplingForm.representativeQuantity" placeholder="如：100.000吨" /></label>
                <label>性质<input v-model="samplingForm.nature" /></label>
                <label>产地<input v-model="samplingForm.origin" /></label>
                <label>生产/进口年限<input v-model="samplingForm.productionYear" /></label>
                <label v-if="samplingSource === 'internal'">入仓/罐时间<input v-model="samplingForm.storageDate" /></label>
                <label v-if="samplingSource === 'internal' && samplingReason === '出库检验'">包装/散装<select v-model="samplingForm.packageType"><option>散装</option><option>包装</option></select></label>
                <label>样品数量<input v-model="samplingForm.sampleCount" /></label>
                <label>样品单位<select v-model="samplingForm.sampleUnit"><option>kg</option><option>g</option><option>份</option><option>L</option></select></label>
                <label>{{ samplingSource === 'external' ? '每仓留样份数' : '留样份数' }}<input v-model="samplingForm.retainCount" /></label>
                <label>{{ samplingSource === 'external' ? '每仓检验样份数' : '检验样份数' }}<input v-model="samplingForm.inspectionCount" /></label>
                <label v-if="samplingSource === 'internal'">保管员<input v-model="samplingForm.keeper" /></label>
                <label>扦样人<input v-model="samplingForm.sampler" /></label>
                <label>扦样日期<input v-model="samplingForm.samplingDate" type="date" /></label>
                <label class="full">相关说明<textarea v-model="samplingForm.remark"></textarea></label>
                <label class="full">{{ samplingSource === 'internal' ? '外部扦样人签名' : '扦样记录' }}<input type="file" /></label>
              </div>
            </div>
            <div class="dialog-actions">
              <button type="button" @click="showSamplingPreview = true">预览扦样登记表</button>
              <button type="button" @click="upsertSamplingRecord('录入中')">保存录入</button>
              <button type="button" @click="upsertSamplingRecord('扦样完成')">提交录入</button>
              <button type="button" @click="closeSamplingForm">取消</button>
            </div>
          </section>
        </div>

        <div v-if="showSamplingDetail" class="process-dialog-mask" @click.self="showSamplingDetail = false">
          <section class="sampling-form-dialog narrow">
            <div class="process-dialog-head"><div><h2>扦样单详情</h2><span>{{ selectedSamplingRecord.orderNo }}</span></div><button type="button" @click="showSamplingDetail = false">关闭</button></div>
            <div class="detail-sections">
              <section class="detail-section">
                <h3>扦样单信息</h3>
                <div class="detail-grid">
                  <div><span>样品编号</span><b>{{ selectedSamplingRecord.sampleNo }}</b></div>
                  <div><span>样品名称</span><b>{{ selectedSamplingRecord.sampleName }}</b></div>
                  <div><span>直属企业</span><b>{{ selectedSamplingRecord.company }}</b></div>
                  <div><span>实际存储库点</span><b>{{ selectedSamplingRecord.depot }}</b></div>
                  <div><span>仓（罐）号</span><b>{{ selectedSamplingRecord.warehouseNo }}</b></div>
                  <div><span>货位</span><b>{{ selectedSamplingRecord.cargoPosition ?? '未登记' }}</b></div>
                  <div v-if="selectedSamplingRecord.reason.includes('春季') || selectedSamplingRecord.reason.includes('秋季')"><span>普查编号</span><b>{{ displaySurveyNo(selectedSamplingRecord) }}</b></div>
                  <div><span>样品品种</span><b>{{ selectedSamplingRecord.grainType }}</b></div>
                  <div><span>扦样人</span><b>{{ selectedSamplingRecord.sampler }}</b></div>
                  <div><span>扦样日期</span><b>{{ selectedSamplingRecord.samplingDate }}</b></div>
                  <div><span>流程状态</span><b>{{ selectedSamplingRecord.status }}</b></div>
                  <div><span>检验事由</span><b>{{ selectedSamplingRecord.reason }}</b></div>
                </div>
              </section>
              <section class="detail-section">
                <h3>送样信息</h3>
                <div class="detail-grid compact">
                  <div><span>送样人</span><b>{{ selectedSamplingRecord.sender ?? '暂未送样' }}</b></div>
                  <div><span>送样时间</span><b>{{ selectedSamplingRecord.sendTime ?? '暂未送样' }}</b></div>
                </div>
              </section>
              <section class="detail-section">
                <h3>收样信息</h3>
                <div class="detail-grid compact">
                  <div><span>收样人</span><b>{{ selectedSamplingRecord.receiver ?? '暂未收样' }}</b></div>
                  <div><span>收样时间</span><b>{{ selectedSamplingRecord.receiveTime ?? '暂未收样' }}</b></div>
                </div>
              </section>
            </div>
          </section>
        </div>

        <div v-if="showSamplingPreview" class="process-dialog-mask" @click.self="showSamplingPreview = false">
          <section class="sampling-preview-dialog">
            <div class="process-dialog-head"><div><h2>扦样登记表预览</h2><span>模板信息自动填充</span></div><button type="button" @click="showSamplingPreview = false">关闭</button></div>
            <div class="sampling-register-preview">
              <div class="preview-brand"><img src="/中储粮logo.png" alt="中储粮" /></div>
              <h2>油脂油料储存过程扦样登记表</h2>
              <div class="preview-sub">直属企业：{{ previewSamplingRecord.company }}　实际储存库点：{{ previewSamplingRecord.depot }}</div>
              <table>
                <tbody>
                  <tr><th>样品编号</th><td>{{ previewSamplingRecord.sampleNo }}</td><th>仓号</th><td>{{ previewSamplingRecord.warehouseNo }}</td><th>样品名称</th><td>{{ previewSamplingRecord.sampleName }}</td></tr>
                  <tr><th>样品数量</th><td>{{ sampleCountWithUnit(previewSamplingRecord) }}</td><th>代表数量/t</th><td>{{ previewSamplingRecord.representativeQuantity ?? '100.000' }}</td><th>性质</th><td>{{ previewSamplingRecord.nature ?? '中央储备粮' }}</td></tr>
                  <tr><th>产地</th><td>{{ previewSamplingRecord.origin ?? '江苏' }}</td><th>生产年限</th><td>{{ previewSamplingRecord.productionYear ?? previewSamplingRecord.samplingDate.slice(0, 4) }}</td><th>入仓时间</th><td>{{ (previewSamplingRecord.storageDate ?? previewSamplingRecord.samplingDate).replace(/-/g, '.') }}</td></tr>
                  <tr><th>扦样目的</th><td>{{ previewSamplingRecord.reason }}</td><th>相关说明</th><td colspan="3">{{ previewSamplingRecord.remark ?? '系统根据当前扦样单录入字段自动填充登记表。' }}</td></tr>
                </tbody>
              </table>
              <div class="preview-sign">保管员签字：{{ previewSamplingRecord.keeper ?? '　　　　' }}　　　　扦样人签字：{{ previewSamplingRecord.sampler }}　　　　扦样日期：{{ previewSamplingRecord.samplingDate.replace(/-/g, '.') }}</div>
            </div>
          </section>
        </div>

        <div v-if="showSamplingLabel" class="process-dialog-mask" @click.self="showSamplingLabel = false">
          <section class="label-preview-dialog wide">
            <div class="process-dialog-head"><div><h2>样品标签打印</h2><span>{{ selectedLabelRecord.sampleNo }}</span></div><button type="button" @click="showSamplingLabel = false">关闭</button></div>
            <div class="sample-label-preview-wrap">
              <div class="label-status-toolbar">
                <button type="button" @click="printSampleLabel"><Printer :size="14" /> {{ labelPrintStatusText(selectedLabelRecord) === '已打印' ? '再次打印' : '打印' }}</button>
              </div>
              <div class="print-sample-label">
                <h2>样品标签</h2>
                <div class="label-row two">
                  <div><span>名称：</span><b>{{ selectedLabelRecord.sampleName }}</b></div>
                  <div><span>编号：</span><b>{{ selectedLabelRecord.sampleNo }}</b></div>
                </div>
                <div class="label-row two">
                  <div><span>来样日期：</span><b>{{ selectedLabelRecord.receiveTime }}</b></div>
                  <div><span>规格：</span><b>{{ sampleCountWithUnit(selectedLabelRecord) }}</b></div>
                </div>
                <div class="label-code-area">
                  <div class="barcode-block" aria-label="样品编号一维码">
                    <img class="barcode-image" :src="barcodeImageUrl(displaySampleNo(selectedLabelRecord))" :alt="`样品编号一维码 ${displaySampleNo(selectedLabelRecord)}`" />
                    <strong>{{ displaySampleNo(selectedLabelRecord) }}</strong>
                  </div>
                  <img class="qr-code-image" :src="qrCodeImageUrl(displaySampleNo(selectedLabelRecord))" :alt="`样品编号二维码 ${displaySampleNo(selectedLabelRecord)}`" />
                </div>
                <p>中央储备粮镇江直属库有限公司（承储）</p>
              </div>
            </div>
          </section>
        </div>

        <div v-if="showInspectionScanDialog" class="process-dialog-mask" @click.self="showInspectionScanDialog = false">
          <section class="inspection-scan-dialog">
            <div class="process-dialog-head"><div><h2>样品扫码录入</h2><span>{{ displaySampleNo(selectedReportRecord) }}</span></div><button type="button" @click="showInspectionScanDialog = false"><X :size="14" /> 关闭</button></div>
            <div class="inspection-scan-body">
              <div class="inspection-scan-target">
                <div><span>待识别样品</span><b>{{ selectedReportRecord.sampleName }}</b></div>
                <div><span>样品编号</span><b>{{ displaySampleNo(selectedReportRecord) }}</b></div>
                <div><span>检验事由</span><b>{{ selectedReportRecord.reason }}</b></div>
              </div>
              <div class="inspection-scan-workspace">
                <div class="inspection-scan-frame">
                  <i class="scan-corner top-left"></i><i class="scan-corner top-right"></i><i class="scan-corner bottom-left"></i><i class="scan-corner bottom-right"></i>
                  <span class="inspection-scan-line"></span>
                  <img :src="qrCodeImageUrl(displaySampleNo(selectedReportRecord))" :alt="`样品二维码 ${displaySampleNo(selectedReportRecord)}`" />
                </div>
                <button type="button" class="scan-read-button" @click="readInspectionSampleCode"><Crosshair :size="15" /> 读取样品标签</button>
              </div>
              <label class="inspection-scan-input">扫码结果<input v-model="inspectionScanValue" placeholder="请扫描或输入样品编号" /></label>
              <div :class="['inspection-scan-result', inspectionScanValue === displaySampleNo(selectedReportRecord) ? 'matched' : inspectionScanValue ? 'mismatched' : 'pending']">
                <CheckCircle2 v-if="inspectionScanValue === displaySampleNo(selectedReportRecord)" :size="15" />
                <TriangleAlert v-else-if="inspectionScanValue" :size="15" />
                <Clock3 v-else :size="15" />
                <span>{{ inspectionScanValue === displaySampleNo(selectedReportRecord) ? '样品编号匹配，可以录入' : inspectionScanValue ? '样品编号不匹配，请重新扫描' : '等待读取样品标签' }}</span>
              </div>
            </div>
            <div class="dialog-actions"><button type="button" :disabled="inspectionScanValue !== displaySampleNo(selectedReportRecord)" @click="confirmInspectionScan">确认扫码录入</button><button type="button" @click="showInspectionScanDialog = false">取消</button></div>
          </section>
        </div>

        <div v-if="showReportForm" class="process-dialog-mask" @click.self="showReportForm = false">
          <section class="report-form-dialog">
            <div class="process-dialog-head">
              <div><h2>录入检验结果</h2><span>{{ selectedReportRecord.sampleNo }}</span></div>
              <button type="button" @click="showReportForm = false">关闭</button>
            </div>
            <div class="report-form-body">
              <div class="report-auto-entry-toolbar">
                <div><b>检化验设备数据</b><span>从仪器台账选择设备并自动获取对应检测指标</span></div>
                <button type="button" @click="openInstrumentDataDialog"><Microscope :size="14" /> 检测数据自动录入</button>
              </div>
              <section class="report-section">
                <h3>基础信息</h3>
                <div class="report-base-grid">
                  <label>样品编号<input :value="selectedReportRecord.sampleNo" readonly /></label>
                  <label>品种<input :value="selectedReportRecord.grainType" readonly /></label>
                  <label>送样单位<input :value="selectedReportRecord.company" readonly /></label>
                  <label>储备性质<input value="中央储备粮" readonly /></label>
                  <label>送样日期<input :value="selectedReportRecord.sendTime ?? selectedReportRecord.receiveTime" readonly /></label>
                  <label>送样人<input :value="selectedReportRecord.sender ?? '—'" readonly /></label>
                  <label>样品规格<input :value="sampleCountWithUnit(selectedReportRecord)" readonly /></label>
                  <label>样品份数<input value="4" readonly /></label>
                  <label>仓（罐）号<input :value="selectedReportRecord.warehouseNo" readonly /></label>
                  <label>代表数量(吨)<input value="100.000" readonly /></label>
                  <label>产地<input value="江苏" readonly /></label>
                  <label>质检报告编号<input :value="selectedReportRecord.reportNo ?? '审批通过后生成'" readonly /></label>
                  <label>检验事由<input :value="selectedReportRecord.reason" readonly /></label>
                  <label>记录人<input value="系统记录员" readonly /></label>
                  <label>检测类别<select v-model="reportMetaForm.category"><option>监督检验</option><option>委托检验</option><option>对比检验</option><option>考核检验</option></select></label>
                  <label>编制人<input v-model="reportMetaForm.compiler" placeholder="请输入" /></label>
                  <label>审核人<input v-model="reportMetaForm.reviewer" placeholder="请输入" /></label>
                  <label>批准人<input v-model="reportMetaForm.approver" placeholder="请输入" /></label>
                  <label class="full">备注<textarea v-model="reportMetaForm.remark" placeholder="请输入备注"></textarea></label>
                </div>
              </section>

              <section class="report-section">
                <h3>质量等级</h3>
                <table class="inspection-result-table">
                  <thead><tr><th>检验项目</th><th>检验依据</th><th>合同要求/检验标准值</th><th>检验值</th><th>单项判定</th><th>检验人</th></tr></thead>
                  <tbody>
                    <tr v-for="item in qualityLevelItems" :key="item.item">
                      <td>{{ item.item }}<span v-if="item.isKey" class="warning-level-badge">关键</span></td>
                      <td>{{ item.type }}</td>
                      <td>{{ item.standard }}</td>
                      <td><div class="result-detect-cell"><input :value="reportResultValue(item.item, 'detectValue')" placeholder="请输入" @input="updateReportResult(item.item, 'detectValue', ($event.target as HTMLInputElement).value)" /><small v-if="reportResultValue(item.item, 'instrumentName')">{{ reportResultValue(item.item, 'instrumentName') }}</small></div></td>
                      <td><input :value="reportResultValue(item.item, 'judgement')" placeholder="自动判定" readonly /></td>
                      <td><select :value="reportResultValue(item.item, 'inspector') || '质检员A'" @change="updateReportResult(item.item, 'inspector', ($event.target as HTMLSelectElement).value)"><option>质检员A</option><option>张三</option><option>李四</option></select></td>
                    </tr>
                  </tbody>
                </table>
              </section>

              <section class="report-section">
                <h3>储存品质</h3>
                <table class="inspection-result-table compact">
                  <thead><tr><th>检验项目</th><th>检验依据</th><th>合同要求/检验标准值</th><th>检验值</th><th>单项判定</th><th>检验人</th></tr></thead>
                  <tbody>
                    <tr v-for="item in storageQualityItems" :key="item.item">
                      <td>{{ item.item }}<span v-if="item.isKey" class="warning-level-badge">关键</span></td><td>{{ item.type }}</td><td>{{ item.standard }}</td><td><div class="result-detect-cell"><input :value="reportResultValue(item.item, 'detectValue')" placeholder="请输入" @input="updateReportResult(item.item, 'detectValue', ($event.target as HTMLInputElement).value)" /><small v-if="reportResultValue(item.item, 'instrumentName')">{{ reportResultValue(item.item, 'instrumentName') }}</small></div></td><td><input :value="reportResultValue(item.item, 'judgement')" placeholder="自动判定" readonly /></td><td><select :value="reportResultValue(item.item, 'inspector') || '质检员A'" @change="updateReportResult(item.item, 'inspector', ($event.target as HTMLSelectElement).value)"><option>质检员A</option><option>张三</option><option>李四</option></select></td>
                    </tr>
                  </tbody>
                </table>
              </section>

              <section class="report-section">
                <h3>卫生指标</h3>
                <table class="inspection-result-table compact">
                  <thead><tr><th>检验项目</th><th>检验依据</th><th>合同要求/检验标准值</th><th>检验值</th><th>单项判定</th><th>检验人</th></tr></thead>
                  <tbody>
                    <tr v-for="item in healthIndicatorItems" :key="item.item">
                      <td>{{ item.item }}<span v-if="item.isKey" class="warning-level-badge">关键</span></td><td>{{ item.type }}</td><td>{{ item.standard }}</td><td><div class="result-detect-cell"><input :value="reportResultValue(item.item, 'detectValue')" placeholder="请输入" @input="updateReportResult(item.item, 'detectValue', ($event.target as HTMLInputElement).value)" /><small v-if="reportResultValue(item.item, 'instrumentName')">{{ reportResultValue(item.item, 'instrumentName') }}</small></div></td><td><input :value="reportResultValue(item.item, 'judgement')" placeholder="自动判定" readonly /></td><td><select :value="reportResultValue(item.item, 'inspector') || '质检员A'" @change="updateReportResult(item.item, 'inspector', ($event.target as HTMLSelectElement).value)"><option>质检员A</option><option>张三</option><option>李四</option></select></td>
                    </tr>
                  </tbody>
                </table>
                <div class="health-extra-grid">
                  <label>质量等级<select v-model="reportMetaForm.qualityGrade"><option value="">请选择</option><option>一等</option><option>二等</option><option>三等</option><option>等外</option><option>不合格</option></select></label>
                  <label>是否达标<select v-model="reportMetaForm.qualified"><option value="">请选择</option><option value="是">是</option><option value="否">否</option></select></label>
                </div>
              </section>
            </div>
            <div class="dialog-actions">
              <button type="button" @click="saveReportResult">保存</button>
              <button type="button" @click="submitReportResult">提交</button>
              <button type="button" @click="showReportForm = false">取消</button>
            </div>
          </section>
        </div>

        <div v-if="showInstrumentDataDialog" class="process-dialog-mask instrument-data-mask" @click.self="showInstrumentDataDialog = false">
          <section class="instrument-data-dialog">
            <div class="process-dialog-head"><div><h2>检测数据自动录入</h2><span>{{ displaySampleNo(selectedReportRecord) }} · {{ selectedReportRecord.grainType }} · {{ selectedReportRecord.reason }}</span></div><button type="button" @click="showInstrumentDataDialog = false"><X :size="14" /> 关闭</button></div>
            <div class="instrument-data-body">
              <div class="instrument-data-summary">
                <div><span>设备台账</span><b>{{ labInstruments.length }} 台</b></div>
                <div><span>已选设备</span><b>{{ selectedInstrumentCodes.length }} 台</b></div>
                <div><span>可填指标</span><b>{{ selectedInstrumentItemCount }} 项</b></div>
              </div>
              <div class="instrument-selection-list">
                <label v-for="row in instrumentAutoEntryRows" :key="row.instrument.code" :class="['instrument-selection-item', { disabled: !row.selectable, selected: selectedInstrumentCodes.includes(row.instrument.code) }]">
                  <input v-model="selectedInstrumentCodes" type="checkbox" :value="row.instrument.code" :disabled="!row.selectable" />
                  <span class="instrument-selection-check"><CheckCircle2 :size="15" /></span>
                  <div class="instrument-selection-main"><b>{{ row.instrument.name }}</b><span>{{ row.instrument.code }} · {{ row.instrument.model }} · {{ row.instrument.location }}</span><small>{{ row.standards.length ? row.standards.map((item) => item.item).join('、') : '当前检验标准无对应检测指标' }}</small></div>
                  <span :class="['lab-status', row.instrument.status === '在用' ? 'ok' : row.instrument.status === '弃用' ? 'danger' : 'warn']">{{ row.instrument.status }}</span>
                </label>
              </div>
            </div>
            <div class="dialog-actions"><span>自动获取的数据仍可在结果表格中人工复核和修改</span><button type="button" :disabled="!selectedInstrumentCodes.length || !selectedInstrumentItemCount" @click="applyInstrumentDetectionData">获取并填充数据</button><button type="button" @click="showInstrumentDataDialog = false">取消</button></div>
          </section>
        </div>

        <div v-if="showConfirmDialog" class="process-dialog-mask system-confirm-mask" @click.self="closeConfirmDialog">
          <section class="system-confirm-dialog" role="alertdialog" aria-modal="true" :aria-labelledby="'system-confirm-title'">
            <div :class="['system-confirm-icon', { danger: confirmDialog.danger }]">
              <TriangleAlert v-if="confirmDialog.danger" :size="22" />
              <CheckCircle2 v-else :size="22" />
            </div>
            <div class="system-confirm-content">
              <h2 id="system-confirm-title">{{ confirmDialog.title }}</h2>
              <p>{{ confirmDialog.message }}</p>
            </div>
            <button class="system-confirm-close" type="button" title="关闭" @click="closeConfirmDialog"><X :size="16" /></button>
            <div class="system-confirm-actions">
              <button type="button" @click="closeConfirmDialog">取消</button>
              <button type="button" :class="{ danger: confirmDialog.danger }" @click="acceptConfirmDialog">{{ confirmDialog.confirmText }}</button>
            </div>
          </section>
        </div>

        <div v-if="showReportView" class="process-dialog-mask" @click.self="showReportView = false">
          <section :class="['report-form-dialog', 'readonly-report-dialog', 'paper-report-dialog', { 'quality-result-dialog': reportViewMode === 'result' }]">
            <div class="process-dialog-head"><div><h2>{{ reportViewMode === 'report' ? '质检报告详情' : '质检结果详情' }}</h2><span>{{ reportViewMode === 'report' ? selectedReportRecord.reportNo : displaySampleNo(selectedReportRecord) }}</span></div><div class="report-view-actions"><button v-if="reportViewMode === 'report'" type="button" @click="printPage"><Printer :size="13" /> 打印</button><button v-if="reportViewMode === 'report'" type="button" @click="exportQualityReport"><Download :size="13" /> 导出</button><button type="button" @click="showReportView = false">关闭</button></div></div>
            <div class="report-form-body readonly-report-body paper-report-body">
              <article :class="['paper-report-page', { 'quality-result-page': reportViewMode === 'result' }]">
                <header class="paper-report-header">
                  <div class="paper-report-brand">
                    <img src="/中储粮logo.png" alt="中储粮" />
                    <span>中央储备粮镇江直属库有限公司</span>
                  </div>
                  <h1>{{ reportViewMode === 'report' ? '质量检验报告' : '质量检验结果' }}</h1>
                  <div class="paper-report-code">
                    <span>{{ reportViewMode === 'report' ? `报告编号：${selectedReportRecord.reportNo}` : `结果状态：${reportApprovalStatusText(selectedReportRecord)}` }}</span>
                    <span>检测类别：{{ selectedReportRecord.reportMeta?.category ?? '监督检验' }}</span>
                  </div>
                </header>

                <table class="paper-info-table">
                  <tbody>
                    <tr><th>样品编号</th><td>{{ displaySampleNo(selectedReportRecord) }}</td><th>样品名称</th><td>{{ selectedReportRecord.sampleName }}</td></tr>
                    <tr><th>粮食品种</th><td>{{ selectedReportRecord.grainType }}</td><th>检验事由</th><td>{{ selectedReportRecord.reason }}</td></tr>
                    <tr><th>送样单位</th><td colspan="3">{{ selectedReportRecord.company }}</td></tr>
                    <tr><th>实际库点</th><td>{{ selectedReportRecord.depot }}</td><th>仓（罐）号</th><td>{{ selectedReportRecord.warehouseNo }}仓</td></tr>
                    <tr><th>样品规格</th><td>{{ sampleCountWithUnit(selectedReportRecord) }}</td><th>代表数量</th><td>{{ selectedReportRecord.representativeQuantity ?? '100.000' }} 吨</td></tr>
                    <tr><th>产地</th><td>{{ selectedReportRecord.origin ?? '江苏' }}</td><th>生产年度</th><td>{{ selectedReportRecord.productionYear ?? selectedReportRecord.samplingDate.slice(0, 4) }}</td></tr>
                    <tr><th>送样日期</th><td>{{ selectedReportRecord.sendTime ?? selectedReportRecord.receiveTime ?? '—' }}</td><th>收样日期</th><td>{{ selectedReportRecord.receiveTime ?? '—' }}</td></tr>
                    <tr v-if="reportViewMode === 'report'"><th>报告结论</th><td><span :class="['paper-result-seal', reportConclusionTone(selectedReportRecord)]">{{ reportConclusion(selectedReportRecord) }}</span></td><th>风险分级</th><td>{{ qualityRiskLevel(selectedReportRecord).label }}</td></tr>
                  </tbody>
                </table>

                <div class="quality-status-banner" aria-label="质量等级与达标状态">
                  <div :class="['quality-status-card', 'grade', qualityGradeTone(selectedReportRecord)]">
                    <span>录入质量等级</span>
                    <strong>{{ qualityGradeText(selectedReportRecord) }}</strong>
                  </div>
                  <div :class="['quality-status-card', 'qualified', qualifiedText(selectedReportRecord) === '是' ? 'pass' : 'fail']">
                    <span>是否达标</span>
                    <strong>{{ qualifiedText(selectedReportRecord) }}</strong>
                  </div>
                </div>

                <section class="paper-report-section">
                  <h2>一、质量等级指标</h2>
                  <table class="paper-result-table">
                    <thead><tr><th>检验项目</th><th>指标属性</th><th>检验依据</th><th>标准值</th><th>检验值</th><th>单项判定</th><th>检验人</th></tr></thead>
                    <tbody>
                      <tr v-for="item in qualityLevelItems" :key="item.item" :class="{ 'paper-unqualified-row': qualityResultValue(selectedReportRecord, item.item) === '不合格' }">
                        <td>{{ item.item }}</td>
                        <td><span :class="['paper-indicator-badge', item.isKey ? 'key' : 'normal']">{{ item.isKey ? '关键指标' : '普通指标' }}</span></td>
                        <td>{{ item.type }}</td>
                        <td>{{ item.standard }}</td>
                        <td>{{ qualityDetectValue(selectedReportRecord, item.item) || '—' }}</td>
                        <td><span :class="['paper-judge', qualityResultValue(selectedReportRecord, item.item) === '不合格' ? 'fail' : 'pass']">{{ qualityResultValue(selectedReportRecord, item.item) }}</span></td>
                        <td>{{ selectedReportRecord.reportResults?.[item.item]?.inspector ?? '质检员A' }}</td>
                      </tr>
                    </tbody>
                  </table>
                </section>

                <section class="paper-report-section">
                  <h2>二、储存品质指标</h2>
                  <table class="paper-result-table">
                    <thead><tr><th>检验项目</th><th>指标属性</th><th>检验依据</th><th>标准值</th><th>检验值</th><th>单项判定</th><th>检验人</th></tr></thead>
                    <tbody>
                      <tr v-for="item in storageQualityItems" :key="item.item" :class="{ 'paper-unqualified-row': qualityResultValue(selectedReportRecord, item.item) === '不合格' }">
                        <td>{{ item.item }}</td>
                        <td><span :class="['paper-indicator-badge', item.isKey ? 'key' : 'normal']">{{ item.isKey ? '关键指标' : '普通指标' }}</span></td>
                        <td>{{ item.type }}</td>
                        <td>{{ item.standard }}</td>
                        <td>{{ qualityDetectValue(selectedReportRecord, item.item) || '—' }}</td>
                        <td><span :class="['paper-judge', qualityResultValue(selectedReportRecord, item.item) === '不合格' ? 'fail' : 'pass']">{{ qualityResultValue(selectedReportRecord, item.item) }}</span></td>
                        <td>{{ selectedReportRecord.reportResults?.[item.item]?.inspector ?? '质检员A' }}</td>
                      </tr>
                    </tbody>
                  </table>
                </section>

                <section class="paper-report-section">
                  <h2>三、卫生指标</h2>
                  <table class="paper-result-table">
                    <thead><tr><th>检验项目</th><th>指标属性</th><th>检验依据</th><th>标准值</th><th>检验值</th><th>单项判定</th><th>检验人</th></tr></thead>
                    <tbody>
                      <tr v-for="item in healthIndicatorItems" :key="item.item" :class="{ 'paper-unqualified-row': healthResultValue(selectedReportRecord, item.item) === '不合格' }">
                        <td>{{ item.item }}</td>
                        <td><span :class="['paper-indicator-badge', item.isKey ? 'key' : 'normal']">{{ item.isKey ? '关键指标' : '普通指标' }}</span></td>
                        <td>{{ item.type }}</td>
                        <td>{{ item.standard }}</td>
                        <td>{{ healthDetectValue(selectedReportRecord, item.item) || '—' }}</td>
                        <td><span :class="['paper-judge', healthResultValue(selectedReportRecord, item.item) === '不合格' ? 'fail' : 'pass']">{{ healthResultValue(selectedReportRecord, item.item) }}</span></td>
                        <td>{{ selectedReportRecord.reportResults?.[item.item]?.inspector ?? '质检员A' }}</td>
                      </tr>
                    </tbody>
                  </table>
                </section>

                <section v-if="reportViewMode === 'report'" class="paper-conclusion-box">
                  <h2>检验结论</h2>
                  <p>{{ reportHasUnqualified(selectedReportRecord) ? `该样品检出不合格指标 ${reportUnqualifiedCount(selectedReportRecord)} 项，其中关键指标 ${qualityRiskLevel(selectedReportRecord).keyCount} 项、普通指标 ${qualityRiskLevel(selectedReportRecord).normalCount} 项，综合判定为${qualityRiskLevel(selectedReportRecord).label}。` : '该样品各项检验指标符合质量要求，综合判定为质量合格。' }}</p>
                  <p v-if="selectedReportRecord.reportMeta?.remark">备注：{{ selectedReportRecord.reportMeta.remark }}</p>
                </section>

                <footer v-if="reportViewMode === 'report'" class="paper-report-signatures">
                  <div><span>编制人</span><b>{{ selectedReportRecord.reportMeta?.compiler ?? '王帅' }}</b></div>
                  <div><span>审核人</span><b>{{ selectedReportRecord.reportMeta?.reviewer ?? '李审核' }}</b></div>
                  <div><span>批准人</span><b>{{ selectedReportRecord.reportMeta?.approver ?? '赵批准' }}</b></div>
                  <div><span>签发日期</span><b>{{ (selectedReportRecord.approvalHistory?.[selectedReportRecord.approvalHistory.length - 1]?.time ?? selectedReportRecord.receiveTime ?? selectedReportRecord.samplingDate).slice(0, 10) }}</b></div>
                </footer>
              </article>
            </div>
          </section>
        </div>

        <div v-if="showApprovalFlow" class="process-dialog-mask" @click.self="showApprovalFlow = false">
          <section class="approval-flow-dialog">
            <div class="process-dialog-head"><div><h2>审批流程图</h2><span>{{ selectedReportRecord.reportNo ?? selectedReportRecord.sampleNo }}</span></div><button type="button" @click="showApprovalFlow = false">关闭</button></div>
            <div class="approval-flow-body">
              <div class="approval-flow-summary">
                <div><span>样品编号</span><b>{{ selectedReportRecord.sampleNo }}</b></div>
                <div><span>当前节点</span><b>{{ selectedReportRecord.approvalStep ?? selectedReportRecord.approvalStatus ?? '未提交' }}</b></div>
                <div><span>样品状态</span><b>{{ selectedReportRecord.labelStatus ?? '待检' }}</b></div>
              </div>
              <div class="approval-route">
                <template v-for="(item, idx) in approvalFlowItems(selectedReportRecord)" :key="item.name">
                  <div v-if="idx > 0" :class="['approval-route-arrow', item.state === 'done' ? 'done' : '']"></div>
                  <div :class="['approval-route-node', item.state]">
                    <span><component :is="item.icon" :size="18" stroke-width="2.2" /></span>
                    <b>{{ item.name }}</b>
                    <small>{{ item.desc }}</small>
                    <em>{{ item.state === 'active' ? '当前节点' : item.state === 'done' ? '已完成' : item.state === 'danger' ? '异常处置' : '未开始' }}</em>
                  </div>
                </template>
              </div>
              <section class="approval-history-card">
                <div class="approval-result-bar">
                  <span>审批前请核对本次质检结果</span>
                  <button type="button" @click="openQualityResultView(selectedReportRecord.sampleNo)"><Eye :size="13" /> 查看质检结果</button>
                </div>
                <div class="history-mode"><span>历史审批意见</span><label><input type="radio" checked /> 正序</label><label><input type="radio" /> 倒序</label></div>
                <div class="approval-timeline">
                  <div v-for="item in [...reportApprovalHistory(selectedReportRecord)].reverse()" :key="`${item.title}-${item.time}`" class="approval-timeline-item">
                    <i></i><b>{{ item.title }}</b><span>{{ item.user }} {{ item.time }}</span><small v-if="item.opinion">审批意见：{{ item.opinion }}</small>
                  </div>
                </div>
                <div v-if="canApproveReport(selectedReportRecord)" class="approval-actions inline">
                  <div>当前节点：{{ selectedReportRecord.approvalStep }}，当前用户：{{ currentLoginUser.name }}（{{ currentLoginUser.role }}）</div>
                  <label class="approval-opinion-editor">审批意见<textarea v-model="approvalOpinion" placeholder="请输入本次审批意见（可选）"></textarea></label>
                  <button type="button" @click="approveReport(selectedReportRecord.sampleNo, 'pass')"><CheckCircle2 :size="13" /> 审批通过/同意</button>
                  <button type="button" @click="approveReport(selectedReportRecord.sampleNo, 'return')"><CornerDownLeft :size="13" /> 不同意退回修改</button>
                </div>
                <div v-else class="approval-tip">当前登录用户无本节点审批权限，仅可查看流程流向与历史审批意见。</div>
              </section>
            </div>
          </section>
        </div>

        <div v-if="showUnqualifiedDialog" class="process-dialog-mask" @click.self="showUnqualifiedDialog = false">
          <section class="unqualified-dialog">
            <div class="process-dialog-head"><div><h2>不合格项明细</h2><span>{{ selectedUnqualifiedRecord.reportNo ?? displaySampleNo(selectedUnqualifiedRecord) }}</span></div><button type="button" @click="showUnqualifiedDialog = false">关闭</button></div>
            <div class="unqualified-dialog-body">
              <div class="unqualified-summary-card">
                <div><span>样品编号</span><b>{{ displaySampleNo(selectedUnqualifiedRecord) }}</b></div>
                <div><span>样品名称</span><b>{{ selectedUnqualifiedRecord.sampleName }}</b></div>
                <div><span>不合格数量</span><b>{{ reportUnqualifiedCount(selectedUnqualifiedRecord) }} 项</b></div>
                <div><span>检验事由</span><b>{{ selectedUnqualifiedRecord.reason }}</b></div>
                <div><span>预警时间</span><b>{{ qualityWarningTime(selectedUnqualifiedRecord) }}</b></div>
              </div>
              <div class="unqualified-detail-wrap">
                <table class="unqualified-detail-table">
                  <colgroup><col class="unqualified-index-col" /><col /><col /><col /><col /></colgroup>
                  <thead><tr><th>序号</th><th>不合格项目</th><th>检测结果</th><th>判定</th><th>处置建议</th></tr></thead>
                  <tbody>
                    <tr v-for="(item, index) in reportUnqualifiedDetails(selectedUnqualifiedRecord)" :key="item.item">
                      <td>{{ index + 1 }}</td><td class="unqualified-text">{{ item.item }}</td><td>{{ item.detectValue }}</td><td><span class="warning-level-badge alarm">不合格</span></td><td>{{ qualityRiskLevel(selectedUnqualifiedRecord).code === 'level1' ? '进入质量报警处置，建议销样或复检' : '进入质量预警，建议复核检测' }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>
        </div>

        <div v-if="showSendSample" class="process-dialog-mask" @click.self="showSendSample = false">
          <section class="send-sample-dialog">
            <div class="process-dialog-head"><div><h2>送样登记</h2><span>{{ selectedSamplingRecord.orderNo }}</span></div><button type="button" @click="showSendSample = false">关闭</button></div>
            <div class="send-form">
              <label>送样确认用户<select v-model="sendSampleForm.sender"><option v-for="user in systemUsers.filter((item) => item.status === '启用')" :key="user.username" :value="user.name">{{ user.name }}（{{ user.role }}）</option></select></label>
              <label>送样时间<input v-model="sendSampleForm.sendTime" type="datetime-local" /></label>
              <button type="button" @click="confirmSendSample">提交送样确认</button>
            </div>
          </section>
        </div>

        <div v-if="showReceiveSample" class="process-dialog-mask" @click.self="showReceiveSample = false">
          <section class="send-sample-dialog">
            <div class="process-dialog-head"><div><h2>收样登记</h2><span>{{ selectedSamplingRecord.orderNo }}</span></div><button type="button" @click="showReceiveSample = false">关闭</button></div>
            <div class="send-form">
              <label>收样确认用户<select v-model="receiveSampleForm.receiver"><option v-for="user in systemUsers.filter((item) => item.status === '启用')" :key="user.username" :value="user.name">{{ user.name }}（{{ user.role }}）</option></select></label>
              <label>收样时间<input v-model="receiveSampleForm.receiveTime" type="datetime-local" /></label>
              <button type="button" @click="confirmReceiveSample">提交收样确认</button>
            </div>
          </section>
        </div>

        <div v-if="showSamplingApprovalFlow && selectedSamplingRecord" class="process-dialog-mask" @click.self="showSamplingApprovalFlow = false">
          <section class="sampling-approval-dialog">
            <div class="process-dialog-head"><div><h2>扦样单送收样审批流</h2><span>{{ selectedSamplingRecord.orderNo }} · {{ selectedSamplingRecord.samplingApprovalStep ?? selectedSamplingRecord.samplingApprovalStatus ?? '已完成' }}</span></div><button type="button" @click="showSamplingApprovalFlow = false"><X :size="13" /> 关闭</button></div>
            <div class="sampling-approval-body">
              <div class="sampling-approval-summary"><div><span>样品名称</span><b>{{ selectedSamplingRecord.sampleName }}</b></div><div><span>当前环节</span><b>{{ selectedSamplingRecord.samplingApprovalStep ?? '流程已确认' }}</b></div><div><span>确认用户</span><b>{{ selectedSamplingRecord.samplingApprovalUser ?? selectedSamplingRecord.receiver ?? selectedSamplingRecord.sender ?? '—' }}</b></div><div><span>当前状态</span><b>{{ selectedSamplingRecord.status }}</b></div></div>
              <div class="sampling-approval-route"><div class="done"><Send :size="16" /><span>提交申请</span></div><i></i><div :class="{ active: selectedSamplingRecord.samplingApprovalStep, done: selectedSamplingRecord.samplingApprovalStatus === '已确认' }"><CheckCircle2 :size="16" /><span>对应用户确认</span></div><i></i><div :class="{ done: selectedSamplingRecord.samplingApprovalStatus === '已确认' }"><PackageCheck :size="16" /><span>状态流转完成</span></div></div>
              <div class="sampling-approval-history"><h3>审批记录</h3><div v-for="item in selectedSamplingRecord.samplingApprovalHistory ?? []" :key="`${item.title}-${item.time}`"><b>{{ item.title }}</b><span>{{ item.user }} · {{ item.time }}</span><p v-if="item.opinion">{{ item.opinion }}</p></div><div v-if="!selectedSamplingRecord.samplingApprovalHistory?.length"><span>暂无审批记录</span></div></div>
              <div v-if="canApproveSamplingTransfer(selectedSamplingRecord)" class="sampling-approval-actions"><label>审批意见<textarea v-model="samplingApprovalOpinion" placeholder="可填写确认说明或退回原因"></textarea></label><div><button type="button" class="return" @click="approveSamplingTransfer('return')"><CornerDownLeft :size="13" /> 退回修改</button><button type="button" class="pass" @click="approveSamplingTransfer('pass')"><CheckCircle2 :size="13" /> 审批并确认</button></div></div><div v-else-if="selectedSamplingRecord.samplingApprovalStep" class="sampling-approval-waiting">当前节点由 {{ selectedSamplingRecord.samplingApprovalUser }} 确认，当前用户仅可查看审批进度。</div>
            </div>
          </section>
        </div>

        <div v-if="upcomingBusinessDialog && upcomingBusinessRecord" class="process-dialog-mask" @click.self="upcomingBusinessDialog = null">
          <section class="send-sample-dialog upcoming-action-dialog">
            <div class="process-dialog-head"><div><h2>{{ upcomingBusinessDialog === 'retain' ? '留样信息核对' : '销样处置信息' }}</h2><span>{{ displaySampleNo(upcomingBusinessRecord) }}</span></div><button type="button" @click="upcomingBusinessDialog = null">关闭</button></div>
            <div class="detail-sections">
              <section class="detail-section"><h3>{{ upcomingBusinessDialog === 'retain' ? '留样信息' : '销样信息' }}</h3><div class="detail-grid compact">
                <div><span>样品名称</span><b>{{ upcomingBusinessRecord.sampleName }}</b></div><div><span>报告编号</span><b>{{ upcomingBusinessRecord.reportNo ?? '未生成' }}</b></div>
                <div><span>样品数量</span><b>{{ sampleCountWithUnit(upcomingBusinessRecord) }}</b></div><div><span>{{ upcomingBusinessDialog === 'retain' ? '留样数量' : '不合格项' }}</span><b>{{ upcomingBusinessDialog === 'retain' ? retainCountWithUnit(upcomingBusinessRecord) : `${reportUnqualifiedCount(upcomingBusinessRecord)} 项` }}</b></div>
                <div><span>{{ upcomingBusinessDialog === 'retain' ? '留样时间' : '销样来源' }}</span><b>{{ upcomingBusinessDialog === 'retain' ? (upcomingBusinessRecord.retainTime ?? ledgerHandleTime(upcomingBusinessRecord)) : (retainStatusLevel(upcomingBusinessRecord) === 'expired' ? '留样到期确认转入' : upcomingBusinessRecord.approvalStatus) }}</b></div><div><span>{{ upcomingBusinessDialog === 'retain' ? '到期状态' : '当前状态' }}</span><b>{{ upcomingBusinessDialog === 'retain' ? retainExpireStatus(upcomingBusinessRecord) : '待销样处置' }}</b></div>
              </div></section>
            </div>
            <div class="dialog-actions"><button v-if="upcomingBusinessDialog === 'retain' && upcomingBusinessRecord.retainRecordId" type="button" @click="openRetainApprovalFlow(upcomingBusinessRecord); upcomingBusinessDialog = null">查看审批流程</button><button v-if="upcomingBusinessDialog === 'destroy' && upcomingBusinessRecord.destroyRecordId" type="button" @click="openDestroyApprovalFlow(upcomingBusinessRecord); upcomingBusinessDialog = null">查看销样流程</button><button v-if="upcomingBusinessDialog === 'destroy' && !upcomingBusinessRecord.destroyRecordId && reportHasFormalReport(upcomingBusinessRecord)" type="button" @click="openReportView(upcomingBusinessRecord.sampleNo); upcomingBusinessDialog = null">查看质检报告</button><button v-else-if="upcomingBusinessDialog === 'destroy' && !upcomingBusinessRecord.destroyRecordId" type="button" @click="openQualityResultView(upcomingBusinessRecord.sampleNo); upcomingBusinessDialog = null">查看质检结果</button><button type="button" @click="upcomingBusinessDialog = null">关闭</button></div>
          </section>
        </div>

        <div v-if="showDestroyForm" class="process-dialog-mask" @click.self="showDestroyForm = false">
          <section class="report-form-dialog retain-entry-dialog destroy-entry-dialog">
            <div class="process-dialog-head"><div><h2>{{ destroyForm.recordId ? '修改销样登记' : '新增销样登记' }}</h2><span>从已留样样品中选择销样对象</span></div><button type="button" @click="showDestroyForm = false"><X :size="14" /> 关闭</button></div>
            <div class="report-form-body retain-entry-body">
              <section class="report-section">
                <h3>选择已留样样品</h3>
                <div class="report-base-grid retain-entry-grid">
                  <label class="full">样品编号<select v-model="destroyForm.sourceRetainId" @change="loadDestroySource"><option value="">请选择已留样样品</option><option v-for="record in destroySourceRecords" :key="record.retainRecordId ?? record.sampleNo" :value="record.retainRecordId ?? record.sampleNo">{{ record.sampleNo }} / {{ record.sampleName }}</option></select></label>
                  <label>样品名称<input v-model="destroyForm.sampleName" readonly /></label>
                  <label>代表仓号<input :value="`${destroyForm.warehouseNo}仓`" readonly /></label>
                  <label>代表货位号<input :value="destroyForm.cargoPosition.replace(`${destroyForm.warehouseNo}仓`, '')" readonly /></label>
                  <label>规格<input v-model="destroyForm.specification" readonly /></label>
                </div>
              </section>
              <section class="report-section">
                <h3>销样申请信息</h3>
                <div class="report-base-grid retain-entry-grid">
                  <label>品种<input v-model="destroyForm.variety" placeholder="请输入品种" /></label>
                  <label>销样来源<select v-model="destroyForm.destroySource"><option>留样到期</option><option>质量不合格</option><option>复检后处置</option><option>其他</option></select></label>
                  <label class="full">备注<textarea v-model="destroyForm.remark" placeholder="请输入销样说明"></textarea></label>
                </div>
              </section>
            </div>
            <div class="dialog-actions"><span>提交后进入审核人、负责人、批准人审批</span><button type="button" :disabled="!destroyForm.sourceRetainId || !destroyForm.variety" @click="saveDestroyForm">提交销样送审</button><button type="button" @click="showDestroyForm = false">取消</button></div>
          </section>
        </div>

        <div v-if="showDestroyApprovalFlow && selectedDestroyRecord" class="process-dialog-mask" @click.self="showDestroyApprovalFlow = false">
          <section class="approval-flow-dialog retain-approval-dialog">
            <div class="process-dialog-head"><div><h2>销样审批流程</h2><span>{{ selectedDestroyRecord.destroyRecordId }}</span></div><button type="button" @click="showDestroyApprovalFlow = false">关闭</button></div>
            <div class="approval-flow-body">
              <div class="approval-flow-summary"><div><span>样品编号</span><b>{{ selectedDestroyRecord.sampleNo }}</b></div><div><span>当前节点</span><b>{{ selectedDestroyRecord.destroyApprovalStep ?? selectedDestroyRecord.destroyApprovalStatus ?? '未送审' }}</b></div><div><span>销样来源</span><b>{{ selectedDestroyRecord.destroySource }}</b></div></div>
              <div class="approval-route">
                <template v-for="(item, idx) in destroyApprovalFlowItems(selectedDestroyRecord)" :key="item.name">
                  <div v-if="idx > 0" :class="['approval-route-arrow', item.state === 'done' ? 'done' : '']"></div>
                  <div :class="['approval-route-node', item.state]"><span><component :is="item.icon" :size="18" stroke-width="2.2" /></span><b>{{ item.name }}</b><small>{{ item.desc }}</small><em>{{ item.state === 'active' ? '当前节点' : item.state === 'done' ? '已完成' : '未开始' }}</em></div>
                </template>
              </div>
              <section class="approval-history-card">
                <div class="history-mode"><span>历史审批意见</span></div>
                <div class="approval-timeline"><div v-for="item in [...(selectedDestroyRecord.destroyApprovalHistory ?? [])].reverse()" :key="`${item.title}-${item.time}`" class="approval-timeline-item"><i></i><b>{{ item.title }}</b><span>{{ item.user }} {{ item.time }}</span><small v-if="item.opinion">审批意见：{{ item.opinion }}</small></div></div>
                <div v-if="canApproveDestroy(selectedDestroyRecord)" class="approval-actions inline"><div>当前节点：{{ selectedDestroyRecord.destroyApprovalStep }}，当前用户：{{ currentLoginUser.name }}（{{ currentLoginUser.role }}）</div><label class="approval-opinion-editor">审批意见<textarea v-model="destroyApprovalOpinion" placeholder="请输入本次审批意见（可选）"></textarea></label><button type="button" @click="approveDestroyRecord('pass')"><CheckCircle2 :size="13" /> {{ selectedDestroyRecord.destroyApprovalStep === '确认销样' ? '确认销样' : '审批通过/同意' }}</button><button type="button" @click="approveDestroyRecord('return')"><CornerDownLeft :size="13" /> 不同意/退回修改</button></div>
                <div v-else class="approval-tip">当前登录用户无本节点审批权限，仅可查看销样流程与历史审批意见。</div>
              </section>
            </div>
          </section>
        </div>

        <div v-if="showRetainScanDialog" class="process-dialog-mask" @click.self="showRetainScanDialog = false">
          <section class="inspection-scan-dialog retain-scan-dialog">
            <div class="process-dialog-head"><div><h2>扫码新增留样</h2><span>扫描样品标签获取留样基础信息</span></div><button type="button" @click="showRetainScanDialog = false"><X :size="14" /> 关闭</button></div>
            <div class="inspection-scan-body">
              <div class="inspection-scan-target">
                <div><span>样品编号</span><b>{{ retainScanRecord?.sampleNo ?? '等待扫描' }}</b></div>
                <div><span>样品名称</span><b>{{ retainScanRecord?.sampleName ?? '—' }}</b></div>
                <div><span>留样数量</span><b>{{ retainScanRecord ? retainCountWithUnit(retainScanRecord) : '—' }}</b></div>
              </div>
              <div class="inspection-scan-workspace">
                <div class="inspection-scan-frame"><span class="scan-corner top-left"></span><span class="scan-corner top-right"></span><span class="scan-corner bottom-left"></span><span class="scan-corner bottom-right"></span><span class="inspection-scan-line"></span><img :src="qrCodeImageUrl(retainScanValue || 'RETAIN-SCAN')" alt="留样标签二维码" /></div>
                <button class="scan-read-button" type="button" @click="readRetainSampleCode"><Crosshair :size="14" /> 读取样品标签</button>
              </div>
              <label class="inspection-scan-input">标签编码<input v-model="retainScanValue" placeholder="扫描或输入样品编号" /></label>
              <div :class="['inspection-scan-result', retainScanExistingRecord ? 'mismatched' : retainScanRecord ? 'matched' : retainScanValue ? 'mismatched' : 'pending']"><TriangleAlert v-if="retainScanExistingRecord" :size="14" /><CheckCircle2 v-else-if="retainScanRecord" :size="14" /><TriangleAlert v-else-if="retainScanValue" :size="14" /><Clock3 v-else :size="14" /><span>{{ retainScanExistingRecord ? '该样品已建立留样记录，请勿重复登记' : retainScanRecord ? '标签识别成功，可进入留样登记' : retainScanValue ? '未找到对应样品标签' : '等待读取样品标签' }}</span></div>
            </div>
            <div class="dialog-actions"><button type="button" :disabled="!retainScanRecord || Boolean(retainScanExistingRecord)" @click="confirmRetainScan">进入新增录入</button><button type="button" @click="showRetainScanDialog = false">取消</button></div>
          </section>
        </div>

        <div v-if="showRetainForm" class="process-dialog-mask" @click.self="showRetainForm = false">
          <section class="report-form-dialog retain-entry-dialog">
            <div class="process-dialog-head"><div><h2>{{ retainForm.recordId ? '修改留样登记' : '新增留样登记' }}</h2><span>{{ retainForm.sampleNo }}</span></div><button type="button" @click="showRetainForm = false"><X :size="14" /> 关闭</button></div>
            <div class="report-form-body retain-entry-body">
              <section class="report-section">
                <h3>扫码获取信息</h3>
                <div class="report-base-grid retain-entry-grid">
                  <label>样品名称<input v-model="retainForm.sampleName" readonly /></label>
                  <label>样品编号<input v-model="retainForm.sampleNo" readonly /></label>
                  <label>来样日期<input v-model="retainForm.sampleDate" readonly /></label>
                  <label>样品数量<input :value="`${retainForm.sampleCount}${retainForm.sampleUnit}`" readonly /></label>
                  <label>代表仓号<input :value="`${retainForm.warehouseNo}仓`" readonly /></label>
                  <label>代表货位号<input :value="retainForm.cargoPosition.replace(`${retainForm.warehouseNo}仓`, '')" readonly /></label>
                  <label class="full">规格<input v-model="retainForm.specification" readonly /></label>
                </div>
              </section>
              <section class="report-section">
                <h3>留样处理信息</h3>
                <div class="report-base-grid retain-entry-grid">
                  <label>过期时间<input v-model="retainForm.expireAt" type="date" /></label>
                  <label>样品状态<input value="审批中" readonly /></label>
                  <label>处理方式<select v-model="retainForm.handlingMethod"><option>留存</option><option>到期销毁</option><option>复检备用</option><option>其他</option></select></label>
                  <label>处置时间<input value="销样审批完成后自动记录" readonly /></label>
                  <label class="full">备注<textarea v-model="retainForm.remark" placeholder="请输入留样说明"></textarea></label>
                </div>
              </section>
            </div>
            <div class="dialog-actions"><span>提交后进入审核人、负责人、批准人审批</span><button type="button" :disabled="!retainForm.expireAt" @click="saveRetainForm">提交留样送审</button><button type="button" @click="showRetainForm = false">取消</button></div>
          </section>
        </div>

        <div v-if="showRetainApprovalFlow && selectedRetainRecord" class="process-dialog-mask" @click.self="showRetainApprovalFlow = false">
          <section class="approval-flow-dialog retain-approval-dialog">
            <div class="process-dialog-head"><div><h2>审批流程</h2><span>{{ selectedRetainRecord.retainRecordId }}</span></div><button type="button" @click="showRetainApprovalFlow = false">关闭</button></div>
            <div class="approval-flow-body">
              <div class="approval-flow-summary"><div><span>样品编号</span><b>{{ selectedRetainRecord.sampleNo }}</b></div><div><span>当前节点</span><b>{{ selectedRetainRecord.retainApprovalStep ?? selectedRetainRecord.retainApprovalStatus ?? '未送审' }}</b></div><div><span>留样状态</span><b>{{ retainWorkflowStatusText(selectedRetainRecord) }}</b></div></div>
              <div class="approval-route">
                <template v-for="(item, idx) in retainApprovalFlowItems(selectedRetainRecord)" :key="item.name">
                  <div v-if="idx > 0" :class="['approval-route-arrow', item.state === 'done' ? 'done' : '']"></div>
                  <div :class="['approval-route-node', item.state]"><span><component :is="item.icon" :size="18" stroke-width="2.2" /></span><b>{{ item.name }}</b><small>{{ item.desc }}</small><em>{{ item.state === 'active' ? '当前节点' : item.state === 'done' ? '已完成' : '未开始' }}</em></div>
                </template>
              </div>
              <section class="approval-history-card">
                <div class="history-mode"><span>历史审批意见</span></div>
                <div class="approval-timeline"><div v-for="item in [...(selectedRetainRecord.retainApprovalHistory ?? [])].reverse()" :key="`${item.title}-${item.time}`" class="approval-timeline-item"><i></i><b>{{ item.title }}</b><span>{{ item.user }} {{ item.time }}</span><small v-if="item.opinion">审批意见：{{ item.opinion }}</small></div></div>
                <div v-if="canApproveRetain(selectedRetainRecord)" class="approval-actions inline"><div>当前节点：{{ selectedRetainRecord.retainApprovalStep }}，当前用户：{{ currentLoginUser.name }}（{{ currentLoginUser.role }}）</div><label class="approval-opinion-editor">审批意见<textarea v-model="retainApprovalOpinion" placeholder="请输入本次审批意见（可选）"></textarea></label><button type="button" @click="approveRetainRecord('pass')"><CheckCircle2 :size="13" /> {{ selectedRetainRecord.retainApprovalStep === '确认留样' ? '确认留样' : '审批通过/同意' }}</button><button type="button" @click="approveRetainRecord('return')"><CornerDownLeft :size="13" /> 不同意/退回修改</button></div>
                <div v-else class="approval-tip">当前登录用户无本节点审批权限，仅可查看审批流程与历史审批意见。</div>
              </section>
            </div>
          </section>
        </div>

        <div v-if="showUserForm" class="process-dialog-mask" @click.self="showUserForm = false">
          <section class="send-sample-dialog">
            <div class="process-dialog-head"><div><h2>{{ userFormMode === 'create' ? '新增用户' : '编辑用户' }}</h2><span>账号与审批角色</span></div><button type="button" @click="showUserForm = false">关闭</button></div>
            <div class="send-form">
              <label>登录账号<input v-model="userForm.username" :readonly="userFormMode === 'edit'" /></label>
              <label>用户姓名<input v-model="userForm.name" /></label>
              <label>登录密码<input v-model="userForm.password" /></label>
              <label>审批角色<select v-model="userForm.role"><option>编制人</option><option>审核人</option><option>负责人</option><option>批准人</option><option>质检员</option></select></label>
              <label>所属部门<input v-model="userForm.department" /></label>
              <label>联系电话<input v-model="userForm.phone" /></label>
              <label>状态<select v-model="userForm.status"><option>启用</option><option>停用</option></select></label>
              <button type="button" @click="saveUser">保存用户</button>
            </div>
          </section>
        </div>
      </main>
    </div>
  </div>
</template>
