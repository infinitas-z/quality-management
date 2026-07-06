<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'
import {
  BookOpenText,
  Boxes,
  BriefcaseBusiness,
  CheckCircle2,
  ChevronDown,
  ChevronRight,
  CircleUserRound,
  CornerDownLeft,
  ClipboardList,
  ClipboardPenLine,
  Edit3,
  Eye,
  Grid2X2,
  Home,
  Layers,
  Menu,
  Microscope,
  PackageCheck,
  Plus,
  Printer,
  Route,
  Search,
  Send,
  Settings,
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
  sampler: string
  samplingDate: string
  source: string
  status: string
  reason: string
  grainType: string
  warehouseNo: string
  company: string
  depot: string
  sender?: string
  sendTime?: string
  receiver?: string
  receiveTime?: string
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
}

interface SamplingForm {
  orderNo: string
  sampleNo: string
  sampleName: string
  sampleCount: string
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
  totalCopies: string
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
}

interface ApprovalHistory {
  title: string
  user: string
  time: string
}

interface ReportResult {
  detectValue: string
  judgement: string
  inspector: string
}

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

interface PersistedState {
  internalSamplingRecords: SamplingRecord[]
  externalSamplingRecords: SamplingRecord[]
  systemUsers: SystemUser[]
  deletedLabelNos: string[]
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
const showProcessDialog = ref(false)
const inspectionTab = ref('overview')
const samplingSource = ref<'internal' | 'external'>('internal')
const samplingReason = ref('入库前检验')
const showSamplingForm = ref(false)
const showSamplingDetail = ref(false)
const showSamplingPreview = ref(false)
const showSamplingLabel = ref(false)
const showSendSample = ref(false)
const showReceiveSample = ref(false)
const showReportForm = ref(false)
const showReportView = ref(false)
const showApprovalFlow = ref(false)

const showInstrumentForm = ref(false)
const instrumentForm = ref({ name: '', model: '', location: '', custodian: '', calibration: '', maintenance: '', usage: 50 })

const showReagentEntryForm = ref(false)
const reagentEntryForm = ref({ name: '', spec: '', unit: '瓶', stock: 0, threshold: 5, location: '', keeper: '', type: '普通试剂' })

const reportFilters = ref({ sampleNo: '', sampleName: '', labelStatus: '', approvalStatus: '' })
const showUnqualifiedDialog = ref(false)
const showUserForm = ref(false)
const selectedSamplingNo = ref('QY20260701001')
const selectedLabelStatus = ref('待检')
const deletedLabelNos = ref<string[]>([])
const loginError = ref('')
const labelFilter = ref({ sampleName: '', source: '', sampleNo: '', labelStatus: '' })
const samplingFilter = ref({ orderNo: '', sampleName: '', reason: '', source: '', status: '' })
const ledgerFilter = ref({ grainType: '', source: '', reason: '', startDate: '', endDate: '' })
const activeQualityReport = ref('cargo')
const labTab = ref('ledger')
const reagentTab = ref('inventory')
const loginForm = ref({ username: 'wangshuai', password: '123456' })
const currentLoginUser = ref<SystemUser | null>(null)
const samplingFormMode = ref<'create' | 'edit'>('create')
const samplingForm = ref<SamplingForm>({ orderNo: '', sampleNo: '', sampleName: '', sampleCount: '2kg × 4', sampler: '张三', samplingDate: '2026-07-04', reason: '入库前检验', grainType: '小麦', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', representativeQuantity: '100.000', nature: '中央储备粮', origin: '江苏', productionYear: '2026', storageDate: '2026-06-17', packageType: '散装', retainCount: '1', inspectionCount: '1', totalCopies: '2', keeper: '王保管', remark: '样品用于质量检验流程演示。', cargoPosition: '49仓1号货位', surveyNo: '' })
const sendSampleForm = ref({ sender: '张三', sendTime: '2026-07-http://localhost:1455/auth/callback?code=ac_y1ze_eM8epyoGjXZtlBbmIutOhHwrc-YAnmGWMFPj0I.28c-z1gY4zNJXvfo3vyaJBINyJbozOkqv3A7TcA3KCc&scope=openid+email+profile+offline_access&state=5e619eceaff9ae282f092744c2da6f5f04T10:30' })
const receiveSampleForm = ref({ receiver: '质检员A', receiveTime: '2026-07-04T11:10' })
const reportMetaForm = ref<ReportMeta>({ category: '监督检验', compiler: '王帅', reviewer: '李审核', approver: '赵批准', remark: '' })
const userFormMode = ref<'create' | 'edit'>('create')
const userForm = ref<UserForm>({ username: '', password: '123456', name: '', role: '质检员', department: '质量管理科', phone: '', status: '启用' })
const storageKey = 'quality-management-demo-state'

const systemUsers = ref<SystemUser[]>([
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

const activeLabelStatuses = ['待检', '在检', '检毕', '留样', '销样']

const editableLabelStatuses = ['待检', '在检']

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
    const matchedLabelStatus = !labelFilter.value.labelStatus || (item.labelStatus ?? '待检') === labelFilter.value.labelStatus
    return matchedStatus && matchedDeleted && matchedName && matchedSource && matchedNo && matchedLabelStatus
  })
})

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
    if (filters.labelStatus && (item.labelStatus ?? '待检') !== filters.labelStatus) return false
    if (filters.approvalStatus) {
      const status = reportApprovalStatusText(item)
      if (!status.includes(filters.approvalStatus)) return false
    }
    return true
  })
})

const selectedReportRecord = computed(() => {
  return reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value) ?? reportRecords.value[0]
})

const selectedUnqualifiedRecord = computed(() => {
  return reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value) ?? reportRecords.value[0]
})

const retainRecords = computed(() => reportRecords.value.filter((item) => item.labelStatus === '留样'))
const destroyRecords = computed(() => reportRecords.value.filter((item) => item.labelStatus === '销样'))

const retainExpireOptions = [30, 60, 90]

const reportUnqualifiedMap: Record<string, string[]> = {
  YP20260701010: ['水分及挥发物(%)', '酸价(KOH)/(mg/g)'],
  YP20260701012: ['水分及挥发物(%)', '黄曲霉毒素B1(μg/kg)', '酸价(KOH)/(mg/g)', '铅(mg/kg)'],
  WYP20260701007: ['铅(mg/kg)'],
}

const reportUnqualifiedItems = (record: SamplingRecord) => {
  const formItems = Object.entries(record.reportResults ?? {}).filter(([, result]) => result.judgement === '不合格').map(([item]) => item)
  if (record.reportResults && formItems.length) return formItems
  if (reportUnqualifiedMap[record.sampleNo]) return reportUnqualifiedMap[record.sampleNo]
  if (record.reportResults) return []
  return []
}

const reportUnqualifiedCount = (record: SamplingRecord) => reportUnqualifiedItems(record).length
const reportHasUnqualified = (record: SamplingRecord) => reportUnqualifiedCount(record) > 0

const openUnqualifiedItems = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  showUnqualifiedDialog.value = true
}

const qualityWarningRecords = computed(() => {
  return reportRecords.value
    .map((record) => {
      const items = reportUnqualifiedItems(record)
      return {
        record,
        items,
        count: items.length,
        level: items.length >= 3 ? '质量报警' : '质量预警',
        status: items.length >= 3 ? '已自动判定为报警' : '已自动提升预警',
      }
    })
    .filter((item) => item.count > 0)
})

const qualityWarningStats = computed(() => {
  const rows = reportRecords.value.map((record) => reportUnqualifiedCount(record))
  return {
    normal: reportRecords.value.filter((record) => record.labelStatus === '留样' && reportUnqualifiedCount(record) === 0).length,
    warning: rows.filter((count) => count > 0 && count < 3).length,
    alarm: rows.filter((count) => count >= 3).length,
  }
})

const currentWarningCount = computed(() => qualityWarningStats.value.warning + qualityWarningStats.value.alarm)

const totalQualityRecords = computed(() => qualityWarningStats.value.normal + qualityWarningStats.value.warning + qualityWarningStats.value.alarm)

const warningRate = computed(() => totalQualityRecords.value ? Math.round((qualityWarningStats.value.warning / totalQualityRecords.value) * 100) : 0)
const alarmRate = computed(() => totalQualityRecords.value ? Math.round((qualityWarningStats.value.alarm / totalQualityRecords.value) * 100) : 0)
const normalRate = computed(() => Math.max(0, 100 - warningRate.value - alarmRate.value))

const warningDonutStyle = computed(() => ({
  background: `conic-gradient(#2f80ed 0 ${normalRate.value}%, #f2994a ${normalRate.value}% ${normalRate.value + warningRate.value}%, #eb5757 ${normalRate.value + warningRate.value}% 100%)`,
}))

const warningWarehouseCount = computed(() => new Set(qualityWarningRecords.value.map((item) => normalizedCargoPosition(item.record))).size)
const warningUnqualifiedTotal = computed(() => qualityWarningRecords.value.reduce((total, item) => total + item.count, 0))

const warningTypes = computed(() => {
  const rows = [
    { label: '质量预警', count: qualityWarningStats.value.warning },
    { label: '质量报警', count: qualityWarningStats.value.alarm },
  ]
  const maxCount = Math.max(...rows.map((item) => item.count), 1)
  return rows.map(({ label, count }) => ({
    label,
    count,
    percent: Math.max(18, Math.round((count / maxCount) * 100)),
  }))
})

const warningRecords = computed(() => qualityWarningRecords.value.map((item) => ({
  level: item.level === '质量报警' ? '报警' : '预警',
  warehouse: `${item.record.warehouseNo}仓`,
  position: normalizedCargoPosition(item.record),
  sampleName: item.record.sampleName,
  content: `不合格项 ${item.count} 项`,
  status: item.status,
  sampleNo: item.record.sampleNo,
})))

const openQualityWarningRecord = (sampleNo: string, warehouse: string) => {
  selectedSamplingNo.value = sampleNo
  selectedWarehouse.value = warehouse
  activeMenu.value = 'warning'
}

const qualityReportTabs = [
  { key: 'cargo', label: '货位明细表' },
  { key: 'warehouse', label: '分货位质量汇总表' },
  { key: 'acceptance', label: '验收申请函' },
  { key: 'season', label: '春秋普检测结果汇总表' },
]

const completedReportRecords = computed(() => reportRecords.value.filter((record) => record.reportNo && record.approvalStatus !== '退回修改' && ['检毕', '留样', '销样'].includes(String(record.labelStatus))))

const displaySampleNo = (record: SamplingRecord) => record.sampleNo?.trim() || record.orderNo || '—'

const cargoPositionOptions = [
  { label: '49仓1号货位', warehouseNo: '49' },
  { label: '49仓2号货位', warehouseNo: '49' },
  { label: '50仓1号货位', warehouseNo: '50' },
  { label: '50仓2号货位', warehouseNo: '50' },
]

const cargoPositionByWarehouse: Record<string, string[]> = cargoPositionOptions.reduce<Record<string, string[]>>((result, item) => {
  result[item.warehouseNo] = [...(result[item.warehouseNo] ?? []), item.label]
  return result
}, {})

const cargoPositionMap: Record<string, string> = {
  '49仓-1号货位': '49仓1号货位',
  '49仓-2号货位': '49仓2号货位',
  '49仓-3号货位': '49仓1号货位',
  '49仓-4号货位': '49仓2号货位',
  '50仓-1号货位': '50仓1号货位',
  '50仓-2号货位': '50仓2号货位',
  '50仓-3号货位': '50仓1号货位',
  '50仓-4号货位': '50仓2号货位',
  '51仓-1号货位': '49仓1号货位',
  '51仓-2号货位': '49仓2号货位',
  '51仓-3号货位': '49仓1号货位',
  '51仓-4号货位': '49仓2号货位',
  '52仓-1号货位': '50仓1号货位',
  '52仓-2号货位': '50仓2号货位',
  '52仓-3号货位': '50仓1号货位',
  '52仓-4号货位': '50仓2号货位',
  '53仓-1号货位': '49仓1号货位',
  '53仓-2号货位': '49仓2号货位',
  '53仓-3号货位': '50仓1号货位',
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

const retainExpireDate = (record: SamplingRecord) => addDaysText(record.retainTime ?? record.handleTime ?? ledgerHandleTime(record), retainExpireDays(record))

const retainExpireStatus = (record: SamplingRecord) => {
  const expireDate = parseDateText(retainExpireDate(record))
  if (!expireDate) return '未设置'
  const today = parseDateText(formatDateText(new Date()))
  if (!today) return '未设置'
  const diffDays = Math.ceil((expireDate.getTime() - today.getTime()) / 86400000)
  if (diffDays < 0) return `已过期 ${Math.abs(diffDays)} 天`
  if (diffDays === 0) return '今日到期'
  return `剩余 ${diffDays} 天`
}

const reportConclusion = (record: SamplingRecord) => {
  const count = reportUnqualifiedCount(record)
  if (count >= 3) return '质量报警'
  if (count >= 1) return '质量预警'
  return '质量合格'
}

const reportQualityGrade = (record: SamplingRecord) => {
  const count = reportUnqualifiedCount(record)
  if (count >= 3) return '不合格'
  if (count >= 1) return '待复核'
  return '一等'
}

const cargoDetailRows = computed(() => completedReportRecords.value.map((record) => ({
  ...record,
  position: normalizedCargoPosition(record),
  warehouseNo: cargoPositionWarehouseNo(normalizedCargoPosition(record)),
  stock: warehouseMarkers.find((item) => item.code === `${cargoPositionWarehouseNo(normalizedCargoPosition(record))}仓`)?.stockQuantity ?? '—',
  conclusion: reportConclusion(record),
  grade: reportQualityGrade(record),
  unqualified: reportUnqualifiedItems(record),
})))

const warehouseSummaryRows = computed(() => {
  const positions = cargoPositionOptions.map((item) => item.label)
  return positions.map((position) => {
  const records = completedReportRecords.value.filter((record) => normalizedCargoPosition(record) === position)
  const warningCount = records.filter((record) => reportUnqualifiedCount(record) > 0 && reportUnqualifiedCount(record) < 3).length
  const alarmCount = records.filter((record) => reportUnqualifiedCount(record) >= 3).length
  const passCount = records.filter((record) => reportUnqualifiedCount(record) === 0).length
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

const labInstruments = [
  { code: 'YQ-CRQ-001', name: '谷物容重器', model: 'GHCS-1000', location: '检化验室A区', custodian: '质检员A', status: '在用', calibration: '2026-08-15', maintenance: '2026-07-10', usage: 86 },
  { code: 'YQ-YD-002', name: '小麦硬度指数测定仪', model: 'JYDB-100', location: '物理检测区', custodian: '王帅', status: '在用', calibration: '2026-09-02', maintenance: '2026-07-12', usage: 72 },
  { code: 'YQ-SF-003', name: '谷物水分测定仪', model: 'LDS-1G', location: '快速检测区', custodian: '质检员A', status: '在用', calibration: '2026-07-18', maintenance: '2026-07-08', usage: 91 },
  { code: 'YQ-ZZ-004', name: '杂质筛选器', model: 'JJSG22×12', location: '样品制备区', custodian: '李审核', status: '待保养', calibration: '2026-10-20', maintenance: '2026-07-05', usage: 58 },
  { code: 'YQ-ZFS-005', name: '脂肪酸值测定仪', model: 'JZSG-Ⅱ', location: '化学检测区', custodian: '赵批准', status: '检定临期', calibration: '2026-07-09', maintenance: '2026-07-14', usage: 64 },
  { code: 'YQ-CS-006', name: '粮食测水仪', model: 'PM-8188', location: '现场快检区', custodian: '质检员A', status: '在用', calibration: '2026-11-12', maintenance: '2026-07-16', usage: 79 },
]

const calibrationRecords = labInstruments.map((item, index) => ({
  instrument: item.name,
  code: item.code,
  planDate: item.calibration,
  agency: index % 2 === 0 ? '镇江市计量测试中心' : '江苏粮油质检站',
  result: item.status === '检定临期' ? '待检定' : '合格',
  certificate: item.status === '检定临期' ? '待上传' : `JD2026${String(index + 1).padStart(3, '0')}`,
}))

const maintenanceRecords = labInstruments.map((item, index) => ({
  instrument: item.name,
  code: item.code,
  date: item.maintenance,
  type: item.status === '待保养' ? '待执行保养' : index % 2 === 0 ? '月度维护' : '清洁校验',
  handler: item.custodian,
  status: item.status === '待保养' ? '待处理' : '已完成',
}))

const environmentRecords = [
  { time: '2026-07-03 08:00', area: '检化验室A区', temperature: 23.6, humidity: 51, status: '正常' },
  { time: '2026-07-03 10:00', area: '物理检测区', temperature: 24.1, humidity: 54, status: '正常' },
  { time: '2026-07-03 12:00', area: '化学检测区', temperature: 25.8, humidity: 61, status: '湿度偏高' },
  { time: '2026-07-03 14:00', area: '快速检测区', temperature: 24.8, humidity: 56, status: '正常' },
  { time: '2026-07-03 16:00', area: '样品制备区', temperature: 26.2, humidity: 63, status: '温湿预警' },
]

const labStats = computed(() => [
  { label: '仪器总数', value: labInstruments.length, tone: 'blue' },
  { label: '在用仪器', value: labInstruments.filter((item) => item.status === '在用').length, tone: 'green' },
  { label: '待检定/保养', value: labInstruments.filter((item) => item.status !== '在用').length, tone: 'orange' },
  { label: '环境异常', value: environmentRecords.filter((item) => item.status !== '正常').length, tone: 'red' },
])

const reagentStocks = [
  { code: 'SJ-PT-001', name: '无水乙醇', type: '普通试剂', spec: '500ml/瓶', stock: 18, unit: '瓶', threshold: 8, location: '普通试剂柜A01', keeper: '质检员A', status: '库存正常' },
  { code: 'SJ-PT-002', name: '氢氧化钠标准溶液', type: '普通试剂', spec: '0.1mol/L', stock: 6, unit: '瓶', threshold: 8, location: '普通试剂柜A02', keeper: '王帅', status: '库存偏低' },
  { code: 'SJ-PT-003', name: '酚酞指示剂', type: '普通试剂', spec: '25g/瓶', stock: 10, unit: '瓶', threshold: 5, location: '普通试剂柜A03', keeper: '李审核', status: '库存正常' },
  { code: 'SJ-YZD-001', name: '盐酸', type: '易制毒试剂', spec: '500ml/瓶', stock: 4, unit: '瓶', threshold: 6, location: '易制毒双人双锁柜B01', keeper: '赵批准 / 质检员A', status: '库存偏低' },
  { code: 'SJ-YZD-002', name: '硫酸', type: '易制毒试剂', spec: '500ml/瓶', stock: 7, unit: '瓶', threshold: 4, location: '易制毒双人双锁柜B02', keeper: '赵批准 / 李审核', status: '重点管控' },
  { code: 'SJ-YZD-003', name: '高锰酸钾', type: '易制毒试剂', spec: '500g/瓶', stock: 3, unit: '瓶', threshold: 5, location: '易制毒双人双锁柜B03', keeper: '赵批准 / 王帅', status: '库存偏低' },
]

const reagentFlowRecords = [
  { time: '2026-07-03 08:30', action: '入库', reagent: '无水乙醇', quantity: '+10瓶', handler: '质检员A', approval: '已入库', remark: '采购到货验收' },
  { time: '2026-07-03 09:15', action: '领用', reagent: '盐酸', quantity: '-1瓶', handler: '王帅', approval: '双人审批通过', remark: '脂肪酸值检测' },
  { time: '2026-07-03 11:20', action: '归还', reagent: '盐酸', quantity: '+0.4瓶', handler: '王帅', approval: '已归还', remark: '剩余试剂回柜' },
  { time: '2026-07-03 14:00', action: '领用', reagent: '氢氧化钠标准溶液', quantity: '-2瓶', handler: '质检员A', approval: '已领用', remark: '酸价检测' },
  { time: '2026-07-03 16:30', action: '盘点', reagent: '高锰酸钾', quantity: '3瓶', handler: '李审核', approval: '库存偏低', remark: '触发采购提醒' },
]

const reagentStats = computed(() => [
  { label: '试剂总类', value: reagentStocks.length, tone: 'blue' },
  { label: '易制毒试剂', value: reagentStocks.filter((item) => item.type === '易制毒试剂').length, tone: 'red' },
  { label: '低库存提醒', value: reagentStocks.filter((item) => item.stock <= item.threshold).length, tone: 'orange' },
  { label: '今日流转', value: reagentFlowRecords.length, tone: 'green' },
])

const lowStockReagents = computed(() => reagentStocks.filter((item) => item.stock <= item.threshold))

const qualityResultValue = (record: SamplingRecord, item: string) => {
  if (record.reportResults?.[item]) return record.reportResults[item].judgement
  if (reportUnqualifiedItems(record).includes(item)) return '不合格'
  return '合格'
}

const qualityDetectValue = (record: SamplingRecord, item: string) => {
  if (record.reportResults?.[item]) return record.reportResults[item].detectValue
  if (!reportUnqualifiedItems(record).includes(item)) return '合格'
  if (item === '水分及挥发物(%)') return '15.8，超标准值'
  if (item === '黄曲霉毒素B1(μg/kg)') return '12.6，超过限量'
  if (item === '酸价(KOH)/(mg/g)') return '4.2，偏高'
  return '合格'
}

const healthResultValue = (record: SamplingRecord, item: string) => {
  if (record.reportResults?.[item]) return record.reportResults[item].judgement
  if (reportUnqualifiedItems(record).includes(item)) return '不合格'
  return '合格'
}

const healthDetectValue = (record: SamplingRecord, item: string) => {
  if (record.reportResults?.[item]) return record.reportResults[item].detectValue
  if (reportUnqualifiedItems(record).includes(item)) return '0.42，超过限量'
  return '未超标'
}

const reportResultValue = (item: string, field: keyof ReportResult) => {
  return selectedReportRecord.value.reportResults?.[item]?.[field] ?? ''
}

const updateReportResult = (item: string, field: keyof ReportResult, value: string) => {
  const record = selectedReportRecord.value
  record.reportResults = {
    ...(record.reportResults ?? {}),
    [item]: {
      detectValue: record.reportResults?.[item]?.detectValue ?? '',
      judgement: record.reportResults?.[item]?.judgement ?? '合格',
      inspector: record.reportResults?.[item]?.inspector ?? '质检员A',
      [field]: value,
    },
  }
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

const qualityLevelItems = ['水分及挥发物(%)', '相对密度D20℃', '过氧化值(g/100g)', '酸价(KOH)/(mg/g)', '不溶性杂质(%)', '气味、滋味', '溶剂残留量(mg/kg)', '黄曲霉毒素B1(μg/kg)', '热损伤粒率(%)', '折光指数', '硬脂酸 C18:0', '亚油酸 C18:2', '棕榈酸 C16:0', '油酸 C18:1']
const healthIndicatorItems = ['总砷(mg/kg)', '铅(mg/kg)']

const internalReasons = ['入库前检验', '入库过程检验', '入库初验', '春季普查', '秋季普查', '存储检验', '出库检验']
const externalReasons = ['入库前检验', '春季普查', '秋季普查', '储存检验', '出库检验', '倒仓后检验']
const grainOptions = ['早籼稻谷', '晚粳稻', '晚籼稻谷', '小麦', '玉米']
const warehouseOptions = ['49', '50']
const flowNodes = computed(() => [
  { name: '扦样单', count: allSamplingRecords.value.length, status: 'active', icon: ClipboardPenLine, tab: 'sampling', action: 'sampling' },
  { name: '样品标签', count: labelRecords.value.length, status: 'done', icon: Printer, tab: 'label', action: 'label' },
  { name: '检验报告', count: completedReportRecords.value.length, status: 'active', icon: ClipboardList, tab: 'report', action: 'report' },
  { name: '在线审批', count: reportRecords.value.filter((item) => item.approvalStep).length, status: 'warning', icon: CheckCircle2, tab: 'report', action: 'approval' },
  { name: '样品台账', count: ledgerRecords.value.length, status: 'done', icon: BookOpenText, tab: 'ledger', action: 'ledger' },
  { name: '留样管理', count: retainRecords.value.length, status: 'normal', icon: PackageCheck, tab: 'retain', action: 'retain' },
  { name: '销样管理', count: destroyRecords.value.length, status: 'normal', icon: Trash2, tab: 'destroy', action: 'destroy' },
])

const openInspectionFlowNode = (tab: string) => {
  activeMenu.value = 'inspection'
  inspectionTab.value = tab
}

const inspectionKpis = computed(() => [
  { label: '今日扦样', value: allSamplingRecords.value.filter((item) => item.samplingDate === '2026-07-04').length },
  { label: '待检样品', value: reportRecords.value.filter((item) => (item.labelStatus ?? '待检') === '待检').length },
  { label: '待审报告', value: reportRecords.value.filter((item) => item.approvalStep).length },
  { label: '留样数量', value: retainRecords.value.length },
  { label: '待销样', value: destroyRecords.value.length },
])

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
  return [...qualityLevelItems, ...healthIndicatorItems].reduce<Record<string, ReportResult>>((result, item) => {
    const isUnqualified = unqualifiedItems.includes(item)
    result[item] = {
      detectValue: isUnqualified ? (item === '水分及挥发物(%)' ? '15.8，超标准值' : item === '黄曲霉毒素B1(μg/kg)' ? '12.6，超过限量' : item === '酸价(KOH)/(mg/g)' ? '4.2，偏高' : '0.42，超过限量') : '合格',
      judgement: isUnqualified ? '不合格' : '合格',
      inspector: '质检员A',
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

const internalSamplingRecords = ref<SamplingRecord[]>([
  { orderNo: 'QY20260701001', sampleNo: 'YP20260701001', sampleName: '49仓小麦样品', sampleCount: '2kg × 4', sampler: '张三', samplingDate: '2026-07-01', source: '内部扦样', status: '扦样完成', reason: '入库前检验', grainType: '小麦', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓1号货位' },
  { orderNo: 'QY20260701002', sampleNo: 'YP20260701002', sampleName: '51仓玉米样品', sampleCount: '1.5kg × 3', sampler: '李四', samplingDate: '2026-07-01', source: '内部扦样', status: '录入中', reason: '存储检验', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓2号货位' },
  { orderNo: 'QY20260701003', sampleNo: 'YP20260701003', sampleName: '53仓复检样品', sampleCount: '2kg × 2', sampler: '王五', samplingDate: '2026-07-02', source: '内部扦样', status: '已送样', reason: '出库检验', grainType: '小麦', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓1号货位', sender: '王五', sendTime: '2026-07-02 14:20' },
  { orderNo: 'QY20260701004', sampleNo: 'YP20260701004', sampleName: '50仓稻谷初验样品', sampleCount: '2kg × 3', sampler: '孙九', samplingDate: '2026-07-03', source: '内部扦样', status: '录入中', reason: '入库初验', grainType: '晚粳稻', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓1号货位' },
  { orderNo: 'QY20260701005', sampleNo: 'YP20260701005', sampleName: '52仓春季普查样品', sampleCount: '1kg × 4', sampler: '钱十', samplingDate: '2026-07-03', source: '内部扦样', status: '扦样完成', reason: '春季普查', grainType: '小麦', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓1号货位' },
  { orderNo: 'QY20260701006', sampleNo: 'YP20260701006', sampleName: '49仓秋季普查样品', sampleCount: '1.5kg × 4', sampler: '吴一', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '待检', reason: '秋季普查', grainType: '早籼稻谷', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓2号货位', sender: '吴一', sendTime: '2026-07-04 10:30', receiver: '质检员A', receiveTime: '2026-07-04 11:10' },
  { orderNo: 'QY20260701007', sampleNo: 'YP20260701007', sampleName: '50仓入库过程检验样品', sampleCount: '2kg × 4', sampler: '刘二', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '在检', approvalStatus: '未提交', reason: '入库过程检验', grainType: '晚粳稻', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓2号货位', sender: '刘二', sendTime: '2026-07-04 09:15', receiver: '质检员A', receiveTime: '2026-07-04 09:50' },
  { orderNo: 'QY20260701008', sampleNo: 'YP20260701008', sampleName: '51仓玉米储存检验样品', sampleCount: '1.5kg × 5', sampler: '陈三', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '检毕', approvalStatus: '审批中', approvalStep: '编制人', reportNo: 'ZJBG-NB-20260701008', reason: '存储检验', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓1号货位', sender: '陈三', sendTime: '2026-07-04 10:05', receiver: '质检员A', receiveTime: '2026-07-04 10:36', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-04 15:05:00' }] },
  { orderNo: 'QY20260701009', sampleNo: 'YP20260701009', sampleName: '52仓春季普查复核样品', sampleCount: '1kg × 6', sampler: '何四', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '检毕', approvalStatus: '审批中', approvalStep: '审核人', reportNo: 'ZJBG20260709', reason: '春季普查', grainType: '小麦', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓2号货位', sender: '何四', sendTime: '2026-07-04 10:45', receiver: '质检员A', receiveTime: '2026-07-04 11:18', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-04 14:10:00' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-04 14:25:00' }] },
  { orderNo: 'QY20260701010', sampleNo: 'YP20260701010', sampleName: '53仓出库检验样品', sampleCount: '2kg × 3', sampler: '周五', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '检毕', approvalStatus: '审批中', approvalStep: '批准人', reportNo: 'ZJBG20260710', reason: '出库检验', grainType: '小麦', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓2号货位', sender: '周五', sendTime: '2026-07-04 13:20', receiver: '质检员A', receiveTime: '2026-07-04 13:48', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-04 15:30:00' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-04 15:42:00' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-04 16:05:00' }] },
  { orderNo: 'QY20260701011', sampleNo: 'YP20260701011', sampleName: '49仓留样演示样品', sampleCount: '1kg × 4', sampler: '郑六', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '留样', approvalStatus: '审批通过', reportNo: 'ZJBG20260711', reason: '秋季普查', grainType: '早籼稻谷', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓1号货位', sender: '郑六', sendTime: '2026-07-04 08:30', receiver: '质检员A', receiveTime: '2026-07-04 09:02', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-04 10:00:00' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-04 10:15:00' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-04 10:36:00' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-04 11:00:00' }] },
  { orderNo: 'QY20260701012', sampleNo: 'YP20260701012', sampleName: '50仓销样演示样品', sampleCount: '2kg × 2', sampler: '冯七', samplingDate: '2026-07-04', source: '内部扦样', status: '已收样', labelStatus: '销样', approvalStatus: '销样', reportNo: 'ZJBG20260712', reason: '入库初验', grainType: '晚粳稻', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓1号货位', sender: '冯七', sendTime: '2026-07-04 11:20', receiver: '质检员A', receiveTime: '2026-07-04 11:45', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-04 13:05:00' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-04 13:18:00' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-04 13:40:00' }, { title: '批准人：不同意，转销样', user: '赵批准（批准人）', time: '2026-07-04 14:05:00' }] },
])

const externalSamplingRecords = ref<SamplingRecord[]>([
  { orderNo: 'WQY20260701001', sampleNo: 'WYP20260701001', sampleName: '50仓外部抽检样品', sampleCount: '2kg × 5', sampler: '赵六', samplingDate: '2026-07-01', source: '外部扦样', status: '扦样完成', reason: '春季普查', grainType: '晚粳稻', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓2号货位' },
  { orderNo: 'WQY20260701002', sampleNo: 'WYP20260701002', sampleName: '52仓外部储存检验样品', sampleCount: '1kg × 4', sampler: '陈七', samplingDate: '2026-07-02', source: '外部扦样', status: '录入中', reason: '储存检验', grainType: '小麦', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓1号货位' },
  { orderNo: 'WQY20260701003', sampleNo: 'WYP20260701003', sampleName: '53仓外部出库检验样品', sampleCount: '2kg × 3', sampler: '郑八', samplingDate: '2026-07-03', source: '外部扦样', status: '已收样', labelStatus: '留样', approvalStatus: '审批通过', reportNo: 'ZJBG20260713', reason: '出库检验', grainType: '小麦', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓1号货位', sender: '郑八', sendTime: '2026-07-03 15:40', receiver: '质检员B', receiveTime: '2026-07-03 16:15', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-03 16:40:00' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-03 16:55:00' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-03 17:10:00' }, { title: '批准人：审批通过/同意', user: '赵批准（批准人）', time: '2026-07-03 17:25:00' }] },
  { orderNo: 'WQY20260701004', sampleNo: 'WYP20260701004', sampleName: '51仓玉米倒仓后检验样品', sampleCount: '1kg × 5', sampler: '周八', samplingDate: '2026-07-04', source: '外部扦样', status: '扦样完成', reason: '倒仓后检验', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓1号货位' },
  { orderNo: 'WQY20260701005', sampleNo: 'WYP20260701005', sampleName: '49仓外部秋普样品', sampleCount: '2kg × 4', sampler: '外检一组', samplingDate: '2026-07-04', source: '外部扦样', status: '已收样', labelStatus: '待检', reason: '秋季普查', grainType: '早籼稻谷', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓2号货位', sender: '外检一组', sendTime: '2026-07-04 12:30', receiver: '质检员B', receiveTime: '2026-07-04 13:05' },
  { orderNo: 'WQY20260701006', sampleNo: 'WYP20260701006', sampleName: '52仓外部退回修改样品', sampleCount: '1.5kg × 4', sampler: '外检二组', samplingDate: '2026-07-04', source: '外部扦样', status: '已收样', labelStatus: '在检', approvalStatus: '退回修改', reportNo: 'ZJBG20260716', reason: '储存检验', grainType: '小麦', warehouseNo: '50', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '50仓2号货位', sender: '外检二组', sendTime: '2026-07-04 14:10', receiver: '质检员B', receiveTime: '2026-07-04 14:45', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-04 15:20:00' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-04 15:35:00' }, { title: '审核人：不同意，退回修改', user: '李审核（审核人）', time: '2026-07-04 15:50:00' }] },
  { orderNo: 'WQY20260701007', sampleNo: 'WYP20260701007', sampleName: '51仓玉米外部批准中样品', sampleCount: '1kg × 5', sampler: '外检三组', samplingDate: '2026-07-04', source: '外部扦样', status: '已收样', labelStatus: '检毕', approvalStatus: '审批中', approvalStep: '批准人', reportNo: 'ZJBG-WB-20260701007', reason: '倒仓后检验', grainType: '玉米', warehouseNo: '49', company: '中央储备粮镇江直属库有限公司', depot: '安鸿智慧粮库', cargoPosition: '49仓2号货位', sender: '外检三组', sendTime: '2026-07-04 15:00', receiver: '质检员B', receiveTime: '2026-07-04 15:32', approvalHistory: [{ title: '提交：送审', user: '王帅（编制人）', time: '2026-07-04 16:00:00' }, { title: '编制人：审批通过/同意', user: '王帅（编制人）', time: '2026-07-04 16:15:00' }, { title: '审核人：审批通过/同意', user: '李审核（审核人）', time: '2026-07-04 16:30:00' }] },
])

const loadPersistedState = () => {
  const raw = localStorage.getItem(storageKey)
  if (!raw) return
  try {
    const state = JSON.parse(raw) as Partial<PersistedState>
    if (Array.isArray(state.internalSamplingRecords)) internalSamplingRecords.value = state.internalSamplingRecords
    if (Array.isArray(state.externalSamplingRecords)) externalSamplingRecords.value = state.externalSamplingRecords
    if (Array.isArray(state.systemUsers)) systemUsers.value = state.systemUsers
    if (Array.isArray(state.deletedLabelNos)) deletedLabelNos.value = state.deletedLabelNos
    allSamplingRecords.value.forEach((record) => {
      record.cargoPosition = normalizeCargoPositionValue(record)
      record.warehouseNo = cargoPositionWarehouseNo(record.cargoPosition)
    })
  } catch {
    localStorage.removeItem(storageKey)
  }
}

const persistState = () => {
  const state: PersistedState = {
    internalSamplingRecords: internalSamplingRecords.value,
    externalSamplingRecords: externalSamplingRecords.value,
    systemUsers: systemUsers.value,
    deletedLabelNos: deletedLabelNos.value,
  }
  localStorage.setItem(storageKey, JSON.stringify(state))
}

onMounted(loadPersistedState)

const handleStorageChange = (event: StorageEvent) => {
  if (event.key === storageKey) loadPersistedState()
}

onMounted(() => window.addEventListener('storage', handleStorageChange))
onUnmounted(() => window.removeEventListener('storage', handleStorageChange))

const warehousePositionTimer = window.setInterval(() => {
  activeWarehousePositionIndex.value = (activeWarehousePositionIndex.value + 1) % selectedWarehousePositions.value.length
}, 3000)

onUnmounted(() => window.clearInterval(warehousePositionTimer))

watch([internalSamplingRecords, externalSamplingRecords, systemUsers, deletedLabelNos], persistState, { deep: true })
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
  const target = labelRecords.value.find((item) => item.sampleNo === sampleNo)
  selectedLabelStatus.value = String(target?.labelStatus ?? '待检')
  showSamplingLabel.value = true
}

const syncLabelStatus = () => {
  const target = allSamplingRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value)
  if (target) {
    target.labelStatus = selectedLabelStatus.value
  }
}

const startInspection = (sampleNo: string) => {
  const target = reportRecords.value.find((item) => item.sampleNo === sampleNo)
  if (target) {
    target.labelStatus = '在检'
    target.approvalStatus = '未提交'
    target.reportResults = target.reportResults ?? buildDefaultReportResults(target)
  }
}

const openReportForm = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  if (canEditReportResult(selectedReportRecord.value)) {
    reportMetaForm.value = selectedReportRecord.value.reportMeta ?? { category: '监督检验', compiler: currentLoginUser.value?.name ?? '王帅', reviewer: '李审核', approver: '赵批准', remark: '' }
    showReportForm.value = true
  }
}

const reportApprovalStatusText = (record: SamplingRecord) => {
  return record.approvalStep ? '审批中' : record.approvalStatus ?? '未提交'
}

const canEditReportResult = (record: SamplingRecord) => {
  return currentLoginUser.value?.role === '编制人' && record.labelStatus === '在检'
}

const addApprovalHistory = (record: SamplingRecord, title: string) => {
  const user = currentLoginUser.value
  record.approvalHistory = [
    ...(record.approvalHistory ?? []),
    { title, user: user ? `${user.name}（${user.role}）` : '系统', time: currentDateTimeText() },
  ]
}

const approvalFlowItems = (record: SamplingRecord) => {
  const history = record.approvalHistory ?? []
  return [
    { name: '提交送审', state: history.some((item) => item.title.includes('提交')) || record.approvalStep || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: Send, desc: '报告提交' },
    { name: '编制人审批', state: record.approvalStep === '编制人' ? 'active' : history.some((item) => item.title.includes('编制人')) || ['审核人', '批准人'].includes(String(record.approvalStep)) || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: Edit3, desc: '结果编制' },
    { name: '审核人审批', state: record.approvalStep === '审核人' ? 'active' : history.some((item) => item.title.includes('审核人')) || record.approvalStep === '批准人' || record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: ClipboardList, desc: '复核报告' },
    { name: '批准人审批', state: record.approvalStep === '批准人' ? 'active' : record.approvalStatus === '审批通过' ? 'done' : 'todo', icon: CheckCircle2, desc: '批准放行' },
    { name: record.labelStatus === '销样' ? '销样管理' : '留样管理', state: record.labelStatus === '留样' || record.labelStatus === '销样' ? record.labelStatus === '销样' ? 'danger' : 'done' : 'todo', icon: record.labelStatus === '销样' ? Trash2 : PackageCheck, desc: record.labelStatus === '销样' ? '异常处置' : '样品留存' },
  ]
}

const openApprovalFlow = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  showApprovalFlow.value = true
}

const addInstrument = () => {
  const form = instrumentForm.value
  const idx = labInstruments.length + 1
  labInstruments.push({
    code: `YQ-XZ-${String(idx).padStart(3, '0')}`,
    name: form.name || `新仪器${idx}`,
    model: form.model || '标准型号',
    location: form.location || '检化验室',
    custodian: form.custodian || '质检员A',
    status: '在用',
    calibration: form.calibration || `2026-${String(12 + idx).padStart(2, '0')}-15`,
    maintenance: form.maintenance || `2026-${String(6 + idx).padStart(2, '0')}-10`,
    usage: form.usage || 50,
  })
  instrumentForm.value = { name: '', model: '', location: '', custodian: '', calibration: '', maintenance: '', usage: 50 }
  showInstrumentForm.value = false
}

const addReagentEntry = () => {
  const form = reagentEntryForm.value
  const idx = reagentStocks.length + 1
  const prefix = form.type === '易制毒试剂' ? 'YZD' : 'PT'
  reagentStocks.push({
    code: `SJ-${prefix}-${String(idx).padStart(3, '0')}`,
    name: form.name || `新试剂${idx}`,
    type: form.type,
    spec: form.spec || '标准规格',
    stock: form.stock || 10,
    unit: form.unit,
    threshold: form.threshold || 5,
    location: form.location || '试剂柜',
    keeper: form.keeper || '质检员A',
    status: (form.stock || 10) <= (form.threshold || 5) ? '库存偏低' : '库存正常',
  })
  reagentEntryForm.value = { name: '', spec: '', unit: '瓶', stock: 0, threshold: 5, location: '', keeper: '', type: '普通试剂' }
  showReagentEntryForm.value = false
}

const saveReportResult = () => {
  const target = reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value)
  if (target && canEditReportResult(target)) {
    target.labelStatus = '在检'
    target.approvalStatus = '未提交'
    target.reportResults = target.reportResults ?? buildDefaultReportResults(target)
    target.reportMeta = { ...reportMetaForm.value }
  }
  showReportForm.value = false
}

const submitReportResult = () => {
  const target = reportRecords.value.find((item) => item.sampleNo === selectedSamplingNo.value)
  if (target && canEditReportResult(target)) {
    target.labelStatus = '检毕'
    target.approvalStatus = '审批中'
    target.approvalStep = '编制人'
    target.reportNo = target.reportNo ?? buildReportNo(target)
    target.reportResults = target.reportResults ?? buildDefaultReportResults(target)
    target.reportMeta = { ...reportMetaForm.value }
    target.approvalHistory = [{ title: '提交：送审', user: `${currentLoginUser.value?.name ?? '编制人'}（编制人）`, time: currentDateTimeText() }]
  }
  showReportForm.value = false
}

const openReportView = (sampleNo: string) => {
  selectedSamplingNo.value = sampleNo
  showReportView.value = true
}

const canApproveReport = (record: SamplingRecord) => {
  return record.labelStatus === '检毕' && record.approvalStep === currentLoginUser.value?.role
}

const currentApprovalTodoRecords = computed(() => reportRecords.value.filter((record) => canApproveReport(record)))

const approveReport = (sampleNo: string, action: 'pass' | 'return' | 'destroy') => {
  selectedSamplingNo.value = sampleNo
  const target = reportRecords.value.find((item) => item.sampleNo === sampleNo)
  if (!target || !canApproveReport(target)) return

  if (action === 'return') {
    addApprovalHistory(target, `${target.approvalStep}：不同意，退回修改`)
    target.labelStatus = '在检'
    target.approvalStatus = '退回修改'
    target.approvalStep = undefined
    target.reportNo = undefined
    target.handleTime = target.approvalHistory?.[target.approvalHistory.length - 1]?.time
    return
  }

  if (action === 'destroy' && target.approvalStep === '批准人') {
    addApprovalHistory(target, '批准人：不同意，转销样')
    target.labelStatus = '销样'
    target.approvalStatus = '销样'
    target.approvalStep = undefined
    target.destroyTime = target.approvalHistory?.[target.approvalHistory.length - 1]?.time
    target.handleTime = target.destroyTime
    return
  }

  if (target.approvalStep === '编制人') {
    addApprovalHistory(target, '编制人：审批通过/同意')
    target.approvalStep = '审核人'
    target.approvalStatus = '审批中'
  } else if (target.approvalStep === '审核人') {
    addApprovalHistory(target, '审核人：审批通过/同意')
    target.approvalStep = '批准人'
    target.approvalStatus = '审批中'
  } else if (target.approvalStep === '批准人') {
    addApprovalHistory(target, '批准人：审批通过/同意')
    target.labelStatus = '留样'
    target.approvalStatus = '审批通过'
    target.approvalStep = undefined
    target.retainExpireDays = target.retainExpireDays ?? 30
    target.retainTime = target.approvalHistory?.[target.approvalHistory.length - 1]?.time
    target.handleTime = target.retainTime
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
  sampleName: '小麦质量检验样品',
  sampleCount: '2kg × 4',
  sampler: '张三',
  samplingDate: '2026-07-04',
  reason: samplingReason.value,
  grainType: '小麦',
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
  totalCopies: '2',
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
  samplingReason.value = samplingSource.value === 'internal' ? internalReasons[0] : externalReasons[0]
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
  const nextRecord: SamplingRecord = { ...records[index], ...samplingForm.value, reason: samplingReason.value, source: samplingSource.value === 'internal' ? '内部扦样' : '外部扦样', status, warehouseNo: cargoPositionWarehouseNo(cargoPosition), cargoPosition }
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
  sendSampleForm.value = { sender: target?.sender ?? '张三', sendTime: (target?.sendTime ?? '2026-07-04 10:30').replace(' ', 'T') }
  showSendSample.value = true
}

const openReceiveSample = (orderNo: string) => {
  selectedSamplingNo.value = orderNo
  const target = findSamplingRecordByOrderNo(orderNo)
  receiveSampleForm.value = { receiver: target?.receiver ?? '质检员A', receiveTime: (target?.receiveTime ?? '2026-07-04 11:10').replace(' ', 'T') }
  showReceiveSample.value = true
}

const confirmSendSample = () => {
  const target = findSamplingRecordByOrderNo(selectedSamplingNo.value)
  if (target) {
    target.status = '已送样'
    target.sender = sendSampleForm.value.sender
    target.sendTime = formatDateTimeValue(sendSampleForm.value.sendTime)
  }
  showSendSample.value = false
}

const confirmReceiveSample = () => {
  const target = findSamplingRecordByOrderNo(selectedSamplingNo.value)
  if (target) {
    target.status = '已收样'
    target.labelStatus = '待检'
    target.receiver = receiveSampleForm.value.receiver
    target.receiveTime = formatDateTimeValue(receiveSampleForm.value.receiveTime)
  }
  showReceiveSample.value = false
}

const closeSamplingForm = () => {
  showSamplingForm.value = false
}

const printPage = () => window.print()

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

const warehouseMarkers = [
  {
    code: '49仓',
    x: 37.8,
    y: 20.0,
    status: 'normal',
    grainType: '小麦',
    stockQuantity: '3,260 吨',
    capacityRate: '87%',
    qualityStatus: '正常',
    qualityLevel: '一等',
    impurity: '0.6%',
    latestTestTime: '2026-06-22',
    warningCount: 0,
    taskStatus: '已完成',
  },
  {
    code: '50仓',
    x: 43.8,
    y: 23.6,
    status: 'warning',
    grainType: '稻谷',
    stockQuantity: '2,980 吨',
    capacityRate: '81%',
    qualityStatus: '预警',
    qualityLevel: '二等',
    impurity: '0.8%',
    latestTestTime: '2026-06-20',
    warningCount: 1,
    taskStatus: '待审核',
  },
  {
    code: '51仓',
    x: 49.0,
    y: 27.2,
    status: 'warning',
    grainType: '玉米',
    stockQuantity: '2,850 吨',
    capacityRate: '82%',
    qualityStatus: '预警',
    qualityLevel: '二等',
    impurity: '0.9%',
    latestTestTime: '2026-06-18',
    warningCount: 2,
    taskStatus: '检验中',
  },
  {
    code: '52仓',
    x: 55.0,
    y: 32.0,
    status: 'normal',
    grainType: '小麦',
    stockQuantity: '3,420 吨',
    capacityRate: '90%',
    qualityStatus: '正常',
    qualityLevel: '一等',
    impurity: '0.5%',
    latestTestTime: '2026-06-25',
    warningCount: 0,
    taskStatus: '已完成',
  },
  {
    code: '53仓',
    x: 59.3,
    y: 35.1,
    status: 'normal',
    grainType: '小麦',
    stockQuantity: '3,120 吨',
    capacityRate: '86%',
    qualityStatus: '正常',
    qualityLevel: '一等',
    impurity: '0.6%',
    latestTestTime: '2026-06-24',
    warningCount: 0,
    taskStatus: '已完成',
  },
]

const selectedWarehouseData = computed(() => {
  return warehouseMarkers.find((item) => item.code === selectedWarehouse.value) ?? warehouseMarkers[0]
})

const warehousePositionRows = (warehouse: typeof warehouseMarkers[number]) => {
  const warehouseNo = warehouse.code.replace('仓', '')
  const total = Number(warehouse.stockQuantity.replace(/[^0-9]/g, ''))
  return [`${warehouseNo}仓1号货位`, `${warehouseNo}仓2号货位`].map((position, index) => {
    const positionRecords = allSamplingRecords.value.filter((record) => normalizedCargoPosition(record) === position)
    const latestRecord = positionRecords.find((record) => record.receiveTime) ?? positionRecords[0]
    const completedRecords = positionRecords.filter((record) => completedReportRecords.value.includes(record))
    const warningCount = completedRecords.filter((record) => reportUnqualifiedCount(record) > 0 && reportUnqualifiedCount(record) < 3).length
    const alarmCount = completedRecords.filter((record) => reportUnqualifiedCount(record) >= 3).length
    const unqualifiedSampleCount = completedRecords.filter((record) => reportUnqualifiedCount(record) > 0).length
    return {
      position,
      status: alarmCount ? 'alarm' : warningCount ? 'warning' : 'normal',
      qualityStatus: alarmCount ? '报警' : warningCount ? '预警' : '正常',
      grainType: latestRecord?.grainType ?? warehouse.grainType,
      stockQuantity: `${Math.round(total * (index === 0 ? 0.52 : 0.48)).toLocaleString()} 吨`,
      qualityLevel: alarmCount ? '待复检' : warningCount ? '待复核' : warehouse.qualityLevel,
      latestTestTime: latestRecord?.receiveTime?.slice(0, 10) ?? latestRecord?.samplingDate ?? warehouse.latestTestTime,
      unqualifiedSampleCount,
    }
  })
}

const warehouseMarkerStatus = (warehouse: typeof warehouseMarkers[number]) => {
  const rows = warehousePositionRows(warehouse)
  if (rows.some((item) => item.status === 'alarm')) return 'alarm'
  if (rows.some((item) => item.status === 'warning')) return 'warning'
  return 'normal'
}

const selectedWarehousePositions = computed(() => {
  return warehousePositionRows(selectedWarehouseData.value)
})

const selectedWarehousePositionData = computed(() => selectedWarehousePositions.value[activeWarehousePositionIndex.value % selectedWarehousePositions.value.length])

const totalStock = computed(() => {
  return warehouseMarkers.reduce((total, item) => total + Number(item.stockQuantity.replace(/[^0-9]/g, '')), 0)
})

const overviewCards = computed(() => [
  { label: '库存总量', value: `${totalStock.value.toLocaleString()} 吨` },
  { label: '仓房总数', value: `${warehouseMarkers.length} 座` },
  { label: '在储品种', value: '3 类' },
  { label: '质量合格率', value: '92.0%' },
  { label: '风险仓房', value: '2 座' },
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
                    <span><i class="wheat"></i>小麦 60%</span>
                    <span><i class="rice"></i>稻谷 20%</span>
                    <span><i class="corn"></i>玉米 20%</span>
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

              <div class="warehouse-map-shell">
                <img class="warehouse-map-image" src="/beijing.png" alt="镇江库区图片" />
                <button
                  v-for="marker in warehouseMarkers"
                  :key="marker.code"
                  class="warehouse-marker"
                  :class="[warehouseMarkerStatus(marker), { selected: selectedWarehouse === marker.code }]"
                  :style="{ left: `${marker.x}%`, top: `${marker.y}%` }"
                  type="button"
                  @click="selectedWarehouse = marker.code"
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
                  <div class="warehouse-position-tabs">
                    <button
                      v-for="(position, index) in selectedWarehousePositions"
                      :key="position.position"
                      type="button"
                      :class="{ active: activeWarehousePositionIndex === index }"
                      @click="activeWarehousePositionIndex = index"
                    >
                      {{ position.position.replace(selectedWarehouseData.code, '') }}
                    </button>
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
                    <div>
                      <span>不合格样品数量</span>
                      <b>{{ selectedWarehousePositionData.unqualifiedSampleCount }} 个</b>
                    </div>
                  </div>
                </aside>
                <div class="map-legend">
                  <span><i class="normal"></i>正常</span>
                  <span><i class="warning"></i>预警</span>
                  <span><i class="alarm"></i>报警</span>
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
                  <span><i class="warning"></i>预警 {{ warningRate }}%</span>
                  <span><i class="alarm"></i>报警 {{ alarmRate }}%</span>
                </div>
              </div>

            

              <div class="warning-summary-grid">
                <div><span>对应货位</span><b>{{ warningWarehouseCount }}</b></div>
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
                  <span :class="['warning-tag', record.level]">{{ record.level }}</span>
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
              <div v-for="item in inspectionKpis" :key="item.label"><span>{{ item.label }}</span><strong>{{ item.value }}</strong></div>
            </div>

            <div class="flow-visual-card">
              <div class="panel-head"><h2>检化验流程总览</h2><span>数据一次录入，样品全程流转</span></div>
              <div class="flow-nodes">
                <template v-for="(node, index) in flowNodes" :key="node.name">
                  <button type="button" class="flow-node" :class="node.status" @click="openInspectionFlowNode(node.tab)">
                    <em>{{ String(index + 1).padStart(2, '0') }}</em>
                    <div class="flow-node-circle"><component :is="node.icon" :size="18" stroke-width="2.2" /></div>
                    <span>{{ node.name }}</span>
                    <b>{{ node.count }}</b>
                  </button>
                  <div v-if="index < flowNodes.length - 1" class="flow-node-arrow" :class="node.status"></div>
                </template>
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
                <div class="panel-head"><h2>近期待办</h2><span>审批与送样</span></div>
                <div class="todo-list">
                  <button v-for="record in currentApprovalTodoRecords" :key="record.sampleNo" type="button" @click="openApprovalFlow(record.sampleNo)"><CheckCircle2 :size="13" /> {{ displaySampleNo(record) }} {{ record.approvalStep }}待审批</button>
                  <button v-if="!currentApprovalTodoRecords.length" type="button"><CheckCircle2 :size="13" /> 当前用户暂无待审批检验报告</button>
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

            <div class="sampling-filter-bar">
              <input v-model="samplingFilter.orderNo" placeholder="扦样单编号" />
              <input v-model="samplingFilter.sampleName" placeholder="样品名称" />
              <select v-model="samplingFilter.reason"><option value="">检验事由</option><option v-for="item in [...internalReasons, ...externalReasons].filter((value, index, array) => array.indexOf(value) === index)" :key="item">{{ item }}</option></select>
              <select v-model="samplingFilter.source"><option value="">样品来源</option><option>内部扦样</option><option>外部扦样</option></select>
              <select v-model="samplingFilter.status"><option value="">流程状态</option><option>录入中</option><option>扦样完成</option><option>已送样</option><option>已收样</option></select>
              <button type="button"><Search :size="14" /> 查询</button>
            </div>

            <div class="sampling-table-card">
              <table class="sampling-table destroy-table">
                <thead>
                  <tr>
                    <th>扦样单编号</th>
                    <th>样品编号</th>
                    <th>样品名称</th>
                    <th>货位</th>
                    <th>样品数量</th>
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
                    <td>{{ record.sampleCount }}</td>
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
              <select v-model="labelFilter.labelStatus">
                <option value="">样品状态</option>
                <option v-for="status in activeLabelStatuses" :key="status">{{ status }}</option>
              </select>
              <button type="button"><Search :size="14" /> 查询</button>
            </div>

            <div class="sampling-table-card">
              <table class="sampling-table destroy-table">
                <thead>
                  <tr>
                    <th>样品编号</th>
                    <th>样品名称</th>
                    <th>样品数量</th>
                    <th>样品来源</th>
                    <th>标签状态</th>
                    <th>来样时间</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="record in labelRecords" :key="record.sampleNo">
                    <td>{{ displaySampleNo(record) }}</td>
                    <td>{{ record.sampleName }}</td>
                    <td>{{ record.sampleCount }}</td>
                    <td>{{ record.source }}</td>
                    <td><span class="sampling-status"><PackageCheck :size="12" /> {{ record.labelStatus ?? '待检' }}</span></td>
                    <td>{{ record.receiveTime }}</td>
                    <td>
                      <div class="sampling-actions">
                        <button type="button" @click="openSampleLabelPrint(record.sampleNo)"><Printer :size="13" /> 打印</button>
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

            <div class="ledger-version-tabs">
              <button class="active" type="button">新版</button>
              <button type="button">旧版</button>
            </div>

            <div class="ledger-sheet-wrap">
              <table class="ledger-sheet-table">
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
                    <td>{{ record.sampleCount }}</td>
                    <td>{{ record.receiveTime?.slice(0, 10) ?? '—' }}</td>
                    <td class="signature-cell">{{ record.receiver ?? '—' }}</td>
                    <td>{{ ledgerHandleMethod(record) }}</td>
                    <td>{{ ledgerApproveUser(record, '批准人') }}</td>
                    <td>{{ ledgerApproveUser(record, '审核人') }}</td>
                    <td>{{ ledgerHandleTime(record) }}</td>
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
                <span>已收样样品自动进入检验报告管理，检测状态按样品编号同步流转</span>
              </div>
              <div class="approval-role-switch readonly">
                <span>当前登录用户</span>
                <b>{{ currentLoginUser.name }} / {{ currentLoginUser.role }}</b>
              </div>
            </div>

            <div class="sampling-filter-bar">
              <input v-model="reportFilters.sampleNo" placeholder="样品编号" />
              <input v-model="reportFilters.sampleName" placeholder="样品名称" />
              <select v-model="reportFilters.labelStatus"><option value="">样品状态</option><option>待检</option><option>在检</option><option>检毕</option><option>留样</option><option>销样</option></select>
              <select v-model="reportFilters.approvalStatus"><option value="">审批状态</option><option>未提交</option><option>审批中</option><option>审批通过</option><option>退回修改</option></select>
            </div>

            <div class="sampling-table-card">
              <table class="sampling-table">
                <thead>
                  <tr>
                    <th>样品编号</th>
                    <th>样品名称</th>
                    <th>样品数量</th>
                    <th>样品来源</th>
                    <th>样品状态</th>
                    <th>审批状态</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="record in filteredReportRecords" :key="record.sampleNo">
                    <td>{{ record.sampleNo }}</td>
                    <td>{{ record.sampleName }}</td>
                    <td>{{ record.sampleCount }}</td>
                    <td>{{ record.source }}</td>
                    <td><span class="sampling-status"><Microscope :size="12" /> {{ record.labelStatus ?? '待检' }}</span></td>
                    <td><span class="sampling-status"><Route :size="12" /> {{ reportApprovalStatusText(record) }}</span></td>
                    <td>
                      <div class="sampling-actions">
                        <button v-if="(record.labelStatus ?? '待检') === '待检'" type="button" @click="startInspection(record.sampleNo)"><Microscope :size="13" /> 开始检测</button>
                        <button v-if="canEditReportResult(record)" type="button" @click="openReportForm(record.sampleNo)"><Edit3 :size="13" /> 录入结果</button>
                        <button v-if="['检毕', '留样', '销样'].includes(String(record.labelStatus))" type="button" @click="openReportView(record.sampleNo)"><Eye :size="13" /> 查看质检报告</button>
                        <button v-if="record.approvalStep" type="button" @click="openApprovalFlow(record.sampleNo)"><Route :size="13" /> 流程图</button>
                        <button v-if="canApproveReport(record)" type="button" @click="approveReport(record.sampleNo, 'pass')"><CheckCircle2 :size="13" /> 同意</button>
                        <button v-if="canApproveReport(record) && currentLoginUser.role !== '编制人'" type="button" @click="approveReport(record.sampleNo, 'return')"><CornerDownLeft :size="13" /> 退回修改</button>
                        <button v-if="canApproveReport(record) && currentLoginUser.role === '批准人'" type="button" @click="approveReport(record.sampleNo, 'destroy')"><Trash2 :size="13" /> 销样</button>
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
                <span>批准人同意通过后，样品信息自动进入留样管理</span>
              </div>
            </div>
            <div class="sampling-table-card">
              <table class="sampling-table retain-table">
                <thead><tr><th>报告编号</th><th>样品编号</th><th>样品名称</th><th>仓号</th><th>品种</th><th>留样状态</th><th>留样期限</th><th>批准时间</th><th>过期时间</th><th>到期状态</th><th>操作</th></tr></thead>
                <tbody>
                  <tr v-for="record in retainRecords" :key="record.sampleNo">
                    <td>{{ record.reportNo ?? '—' }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.warehouseNo }}</td><td>{{ record.grainType }}</td><td><span class="sampling-status"><PackageCheck :size="12" /> 留样</span></td><td><select v-model.number="record.retainExpireDays" class="retain-expire-select"><option v-for="days in retainExpireOptions" :key="days" :value="days">{{ days }}天</option></select></td><td>{{ ledgerHandleTime(record) }}</td><td>{{ retainExpireDate(record) }}</td><td><span class="retain-expire-status">{{ retainExpireStatus(record) }}</span></td>
                    <td><div class="sampling-actions"><button type="button" @click="openReportView(record.sampleNo)"><Eye :size="13" /> 查看质检报告</button></div></td>
                  </tr>
                  <tr v-if="!retainRecords.length"><td colspan="11">暂无留样数据，批准人审批通过后自动生成。</td></tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else-if="inspectionTab === 'destroy'" class="sampling-page">
            <div class="sampling-toolbar">
              <div class="module-title-block">
                <h2>销样管理</h2>
                <span>批准人选择销样后，样品信息自动进入销样管理</span>
              </div>
            </div>
            <div class="sampling-table-card">
              <table class="sampling-table">
                <thead><tr><th>报告编号</th><th>样品编号</th><th>样品名称</th><th>仓号</th><th>品种</th><th>不合格项</th><th>销样来源</th><th>操作</th></tr></thead>
                <tbody>
                  <tr v-for="record in destroyRecords" :key="record.sampleNo">
                    <td>{{ record.reportNo }}</td><td>{{ displaySampleNo(record) }}</td><td>{{ record.sampleName }}</td><td>{{ record.warehouseNo }}</td><td>{{ record.grainType }}</td><td><button class="unqualified-count-btn" type="button" @click="openUnqualifiedItems(record.sampleNo)">{{ reportUnqualifiedCount(record) }} 项</button></td><td>{{ record.approvalStatus }}</td>
                    <td><div class="sampling-actions"><button type="button" @click="openReportView(record.sampleNo)"><Eye :size="13" /> 查看质检报告</button></div></td>
                  </tr>
                  <tr v-if="!destroyRecords.length"><td colspan="8">暂无销样数据。</td></tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else class="coming-panel">
            <h2>{{ inspectionTabs.find((item) => item.key === inspectionTab)?.label }}</h2>
            <p>该模块后续继续补充，目前先完成流程总览与扦样单管理。</p>
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
              <div><b>正常质量检验</b><strong>{{ qualityWarningStats.normal }} 条</strong><span>0项不合格，不进入预警记录</span></div>
              <div><b>质量预警</b><strong>{{ qualityWarningStats.warning }} 条</strong><span>1-2项不合格，可查看对应质检报告</span></div>
              <div><b>质量报警</b><strong>{{ qualityWarningStats.alarm }} 条</strong><span>3项及以上不合格，进入报警处置</span></div>
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
                    <th>不合格项数</th>
                    <th>不合格内容</th>
                    <th>判定说明</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in qualityWarningRecords" :key="item.record.sampleNo">
                    <td><span :class="['warning-level-badge', item.level === '质量报警' ? 'alarm' : 'warning']">{{ item.level }}</span></td>
                    <td>{{ item.record.reportNo ?? '—' }}</td>
                    <td>{{ displaySampleNo(item.record) }}</td>
                    <td>{{ item.record.sampleName }}</td>
                    <td>{{ normalizedCargoPosition(item.record) }}</td>
                    <td>{{ item.record.grainType }}</td>
                    <td><button class="unqualified-count-btn" type="button" @click="openUnqualifiedItems(item.record.sampleNo)">{{ item.count }} 项</button></td>
                    <td><button class="unqualified-detail-link" type="button" @click="openUnqualifiedItems(item.record.sampleNo)">查看不合格项</button></td>
                    <td>{{ item.status }}</td>
                    <td><div class="sampling-actions"><button type="button" @click="openReportView(item.record.sampleNo)"><Eye :size="13" /> 查看质检报告</button></div></td>
                  </tr>
                  <tr v-if="!qualityWarningRecords.length"><td colspan="10">暂无质量预警记录，当前检验报告均为正常质量检验。</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <section v-else-if="activeMenu === 'report'" class="inspection-page">
          <div class="quality-report-page">
            <div class="quality-report-hero">
              <div>
                <span>系统自动生成</span>
                <h2>质量报表管理</h2>
                <p>基于检验流程管理中的扦样、收样、检验报告、审批与预警结果自动汇总生成。</p>
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
              <div class="report-sheet-head"><div><h3>货位明细表</h3><span>自动生成编号：ZL-HW-20260703</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <div class="paper-report-title"><h2>中央储备粮镇江直属库有限公司货位明细表</h2><p>统计期间：2026-07-01 至 2026-07-03　制表部门：质量管理科　数据来源：检验流程管理系统</p></div>
              <table class="visual-report-table">
                <thead><tr><th>样品编号</th><th>货位</th><th>仓号</th><th>品种</th><th>库存数量</th><th>样品数量</th><th>检验事由</th><th>质量等级</th><th>检验结论</th><th>风险项</th></tr></thead>
                <tbody>
                  <tr v-for="row in cargoDetailRows" :key="row.sampleNo">
                    <td>{{ displaySampleNo(row) }}</td><td>{{ row.position }}</td><td>{{ row.warehouseNo }}仓</td><td>{{ row.grainType }}</td><td>{{ row.stock }}</td><td>{{ row.sampleCount }}</td><td>{{ row.reason }}</td><td>{{ row.grade }}</td><td><span :class="['report-result-pill', row.conclusion === '质量合格' ? 'ok' : row.conclusion === '质量预警' ? 'warning' : 'alarm']">{{ row.conclusion }}</span></td><td>{{ row.unqualified.length ? row.unqualified.join('、') : '无' }}</td>
                  </tr>
                </tbody>
              </table>
              </div>

            <div class="quality-report-sheet" v-else-if="activeQualityReport === 'warehouse'">
              <div class="report-sheet-head"><div><h3>分货位质量汇总表</h3><span>自动生成编号：ZL-FHW-20260703</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <div class="paper-report-title"><h2>分货位质量汇总表</h2><p>汇总口径：按仓房货位自动汇总　制表部门：质量管理科　合格率根据质检报告不合格项自动计算</p></div>
              <table class="visual-report-table warehouse-report-table">
                <thead><tr><th>货位/仓号</th><th>样品数量</th><th>合格数量</th><th>预警数量</th><th>报警数量</th><th>合格率</th><th>主要风险项</th><th>处理建议</th></tr></thead>
                <tbody>
                  <tr v-for="row in warehouseSummaryRows" :key="row.position">
                    <td>{{ row.position }}</td><td>{{ row.sampleCount }}</td><td>{{ row.passCount }}</td><td>{{ row.warningCount }}</td><td>{{ row.alarmCount }}</td><td>{{ row.passRate }}</td><td>{{ row.mainIssue }}</td><td>{{ row.alarmCount ? '启动报警处置并复检' : row.warningCount ? '关注趋势，安排复核' : '持续正常监测' }}</td>
                  </tr>
                </tbody>
              </table>
            </div>

            <div class="quality-report-sheet" v-else-if="activeQualityReport === 'acceptance'">
              <div class="report-sheet-head"><div><h3>验收申请函</h3><span>自动提取入库类检验样品生成验收申请明细</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <div class="acceptance-letter">
                <h3>中央储备粮镇江直属库有限公司验收申请函</h3>
                <div class="paper-letter-meta"><span>编号：YS-SQ-20260703</span><span>申请部门：质量管理科</span><span>申请日期：2026-07-03</span></div>
                <p>根据检验流程管理系统数据，以下入库相关样品已完成收样、检验及报告生成，现申请组织质量验收。</p>
                <table class="visual-report-table"><thead><tr><th>报告编号</th><th>样品编号</th><th>仓号</th><th>品种</th><th>检验事由</th><th>检验结论</th><th>申请意见</th></tr></thead><tbody><tr v-for="row in acceptanceRows" :key="row.sampleNo"><td>{{ row.reportNo ?? '待生成' }}</td><td>{{ displaySampleNo(row) }}</td><td>{{ row.warehouseNo }}仓</td><td>{{ row.grainType }}</td><td>{{ row.reason }}</td><td>{{ reportConclusion(row) }}</td><td>{{ reportUnqualifiedCount(row) ? '建议复核后验收' : '建议通过验收' }}</td></tr></tbody></table>
                </div>
            </div>

            <div class="quality-report-sheet" v-else>
              <div class="report-sheet-head"><div><h3>春秋普检测结果汇总表</h3><span>自动汇总春季普查、秋季普查样品的检测结果</span></div><button type="button" @click="printPage"><Printer :size="13" /> 打印</button></div>
              <div class="paper-report-title"><h2>春秋普检测结果汇总表</h2><p>统计期间：2026年度春秋普查　检测单位：中央储备粮镇江直属库有限公司检化验室</p></div>
              <table class="visual-report-table season-report-table"><thead><tr><th>样品编号</th><th>普查编号</th><th>普查类型</th><th>仓号</th><th>品种</th><th>水分</th><th>酸价</th><th>黄曲霉毒素B1</th><th>卫生指标</th><th>综合判定</th></tr></thead><tbody><tr v-for="row in seasonRows" :key="row.sampleNo"><td>{{ displaySampleNo(row) }}</td><td>{{ displaySurveyNo(row) }}</td><td>{{ row.reason }}</td><td>{{ row.warehouseNo }}仓</td><td>{{ row.grainType }}</td><td :class="{ 'unqualified-text': qualityResultValue(row, '水分及挥发物(%)') === '不合格' }">{{ qualityDetectValue(row, '水分及挥发物(%)') }}</td><td :class="{ 'unqualified-text': qualityResultValue(row, '酸价(KOH)/(mg/g)') === '不合格' }">{{ qualityDetectValue(row, '酸价(KOH)/(mg/g)') }}</td><td :class="{ 'unqualified-text': qualityResultValue(row, '黄曲霉毒素B1(μg/kg)') === '不合格' }">{{ qualityDetectValue(row, '黄曲霉毒素B1(μg/kg)') }}</td><td>{{ healthDetectValue(row, '铅(mg/kg)') }}</td><td>{{ reportConclusion(row) }}</td></tr></tbody></table>
            </div>
          </div>
        </section>

        <section v-else-if="activeMenu === 'lab'" class="inspection-page">
          <div class="lab-page">
            <div class="lab-hero">
              <div><span>检化验室数字化</span><h2>仪器管理与温湿度自动采集</h2></div>
              <div class="lab-stat-grid"><div v-for="item in labStats" :key="item.label" :class="['lab-stat-card', item.tone]"><b>{{ item.value }}</b><span>{{ item.label }}</span></div></div>
            </div>

            <div class="inspection-tabs lab-module-tabs">
              <button type="button" :class="{ active: labTab === 'ledger' }" @click="labTab = 'ledger'">仪器台账与温湿度</button>
              <button type="button" :class="{ active: labTab === 'calibration' }" @click="labTab = 'calibration'">检定校准记录</button>
              <button type="button" :class="{ active: labTab === 'maintenance' }" @click="labTab = 'maintenance'">维护保养记录</button>
            </div>

            <div v-if="labTab === 'ledger'" class="lab-main-grid lab-ledger-grid">
              <section class="lab-panel instrument-panel">
                <div class="lab-panel-head"><div><h3>检化验仪器台账</h3><span>仪器基础信息、位置、负责人和使用状态电子化管理</span></div><button type="button" @click="showInstrumentForm = true"><Plus :size="13" /> 新增仪器</button></div>
                <div class="instrument-card-grid">
                  <article v-for="item in labInstruments" :key="item.code" class="instrument-card">
                    <div class="instrument-card-head"><b>{{ item.name }}</b><span :class="{ warn: item.status !== '在用' }">{{ item.status }}</span></div>
                    <p>{{ item.code }} / {{ item.model }}</p>
                    <div class="instrument-meta"><span>位置：{{ item.location }}</span><span>责任人：{{ item.custodian }}</span><span>下次检定：{{ item.calibration }}</span></div>
                    <div class="instrument-usage"><i :style="{ width: `${item.usage}%` }"></i></div>
                  </article>
                </div>
              </section>

              <section class="lab-panel env-panel">
                <div class="lab-panel-head"><div><h3>温湿度自动采集</h3><span>采集频率：每 2 小时 / 数据自动入库</span></div></div>
                <div class="env-gauge-row"><div><b>24.9℃</b><span>平均温度</span></div><div><b>57%</b><span>平均湿度</span></div><div><b>2</b><span>异常记录</span></div></div>
                <div class="env-record-list"><div v-for="item in environmentRecords" :key="`${item.area}-${item.time}`"><span>{{ item.time }}</span><b>{{ item.area }}</b><em>{{ item.temperature }}℃ / {{ item.humidity }}%</em><strong :class="{ warn: item.status !== '正常' }">{{ item.status }}</strong></div></div>
              </section>
            </div>

            <section v-else-if="labTab === 'calibration'" class="lab-panel lab-single-module">
              <div class="lab-panel-head"><div><h3>检定校准记录</h3><span>按计划提醒检定，到期状态自动标识</span></div></div>
              <table class="lab-table"><thead><tr><th>仪器名称</th><th>仪器编号</th><th>计划检定日期</th><th>检定机构</th><th>结果</th><th>证书编号</th></tr></thead><tbody><tr v-for="item in calibrationRecords" :key="item.code"><td>{{ item.instrument }}</td><td>{{ item.code }}</td><td>{{ item.planDate }}</td><td>{{ item.agency }}</td><td><span :class="['lab-status', item.result === '合格' ? 'ok' : 'warn']">{{ item.result }}</span></td><td>{{ item.certificate }}</td></tr></tbody></table>
            </section>

            <section v-else class="lab-panel lab-single-module">
              <div class="lab-panel-head"><div><h3>维护保养记录</h3><span>清洁、维护、故障处理和保养计划闭环管理</span></div></div>
              <table class="lab-table"><thead><tr><th>仪器名称</th><th>仪器编号</th><th>维护日期</th><th>维护类型</th><th>处理人</th><th>状态</th></tr></thead><tbody><tr v-for="item in maintenanceRecords" :key="`${item.code}-${item.date}`"><td>{{ item.instrument }}</td><td>{{ item.code }}</td><td>{{ item.date }}</td><td>{{ item.type }}</td><td>{{ item.handler }}</td><td><span :class="['lab-status', item.status === '已完成' ? 'ok' : 'warn']">{{ item.status }}</span></td></tr></tbody></table>
            </section>
          </div>

          <div v-if="showInstrumentForm" class="process-dialog-mask" @click.self="showInstrumentForm = false">
            <section class="send-sample-dialog">
              <div class="process-dialog-head"><div><h2>新增仪器</h2><span>填写仪器基本信息</span></div><button type="button" @click="showInstrumentForm = false"><X :size="14" /> 关闭</button></div>
              <div class="send-form">
                <label>仪器名称<input v-model="instrumentForm.name" placeholder="如：谷物水分测定仪" /></label>
                <label>规格型号<input v-model="instrumentForm.model" placeholder="如：LDS-1G" /></label>
                <label>存放位置<input v-model="instrumentForm.location" placeholder="如：检化验室A区" /></label>
                <label>责任人<input v-model="instrumentForm.custodian" placeholder="如：质检员A" /></label>
                <label>下次检定日期<input v-model="instrumentForm.calibration" type="date" /></label>
                <label>下次保养日期<input v-model="instrumentForm.maintenance" type="date" /></label>
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

            <div class="sampling-toolbar" style="margin-top:16px">
              <div class="sampling-source-tabs">
                <button type="button" :class="{ active: reagentTab === 'inventory' }" @click="reagentTab = 'inventory'">试剂库存台账</button>
                <button type="button" :class="{ active: reagentTab === 'records' }" @click="reagentTab = 'records'">出入库与领还记录</button>
              </div>
            </div>

            <div v-if="reagentTab === 'inventory'" class="reagent-main-grid">
              <section class="lab-panel">
                <div class="lab-panel-head"><div><h3>试剂库存台账</h3><span>普通试剂与易制毒试剂分类管理，库存低自动标识</span></div><button type="button" @click="showReagentEntryForm = true"><Plus :size="13" /> 入库登记</button></div>
                <table class="lab-table reagent-table"><thead><tr><th>试剂编号</th><th>试剂名称</th><th>类别</th><th>规格</th><th>库存</th><th>预警阈值</th><th>存放位置</th><th>保管人</th><th>状态</th></tr></thead><tbody><tr v-for="item in reagentStocks" :key="item.code"><td>{{ item.code }}</td><td>{{ item.name }}</td><td><span :class="['reagent-type', item.type === '易制毒试剂' ? 'danger' : 'normal']">{{ item.type }}</span></td><td>{{ item.spec }}</td><td><b :class="{ 'unqualified-text': item.stock <= item.threshold }">{{ item.stock }}{{ item.unit }}</b></td><td>{{ item.threshold }}{{ item.unit }}</td><td>{{ item.location }}</td><td>{{ item.keeper }}</td><td><span :class="['lab-status', item.stock <= item.threshold || item.type === '易制毒试剂' ? 'warn' : 'ok']">{{ item.status }}</span></td></tr></tbody></table>
              </section>

              <section class="lab-panel">
                <div class="lab-panel-head"><div><h3>低库存采购提醒</h3><span>低于阈值时自动生成采购建议</span></div></div>
                <div class="purchase-alert-list"><div v-for="item in lowStockReagents" :key="item.code"><b>{{ item.name }}</b><span>{{ item.type }} / 当前 {{ item.stock }}{{ item.unit }}</span><em>建议采购 {{ Math.max(item.threshold * 2 - item.stock, item.threshold) }}{{ item.unit }}</em></div></div>
              </section>
            </div>

            <div v-if="reagentTab === 'records'" class="lab-panel" style="margin-top:0">
              <div class="lab-panel-head"><div><h3>出入库与领还记录</h3><span>入库、领用、归还、盘点全过程留痕</span></div></div>
              <table class="lab-table"><thead><tr><th>时间</th><th>操作</th><th>试剂名称</th><th>数量</th><th>经办人</th><th>审批/状态</th><th>备注</th></tr></thead><tbody><tr v-for="item in reagentFlowRecords" :key="`${item.time}-${item.reagent}`"><td>{{ item.time }}</td><td>{{ item.action }}</td><td>{{ item.reagent }}</td><td>{{ item.quantity }}</td><td>{{ item.handler }}</td><td><span :class="['lab-status', item.approval.includes('偏低') ? 'warn' : 'ok']">{{ item.approval }}</span></td><td>{{ item.remark }}</td></tr></tbody></table>
            </div>
          </div>

          <div v-if="showReagentEntryForm" class="process-dialog-mask" @click.self="showReagentEntryForm = false">
            <section class="send-sample-dialog">
              <div class="process-dialog-head"><div><h2>入库登记</h2><span>新增试剂入库</span></div><button type="button" @click="showReagentEntryForm = false"><X :size="12" /> 关闭</button></div>
              <div class="send-form">
                <label>试剂名称<input v-model="reagentEntryForm.name" placeholder="如：无水乙醇" /></label>
                <label>规格<input v-model="reagentEntryForm.spec" placeholder="如：500ml/瓶" /></label>
                <label>单位<select v-model="reagentEntryForm.unit"><option>瓶</option><option>L</option><option>kg</option><option>g</option></select></label>
                <label>入库数量<input v-model.number="reagentEntryForm.stock" type="number" min="0" /></label>
                <label>预警阈值<input v-model.number="reagentEntryForm.threshold" type="number" min="0" /></label>
                <label>存放位置<input v-model="reagentEntryForm.location" placeholder="如：试剂柜A01" /></label>
                <label>保管人<input v-model="reagentEntryForm.keeper" placeholder="如：质检员A" /></label>
                <label>类别<select v-model="reagentEntryForm.type"><option>普通试剂</option><option>易制毒试剂</option></select></label>
                <button type="button" @click="addReagentEntry">确认入库</button>
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
              <p>检验报告提交后进入“待编制审批”，只有编制人账号可审批；编制通过后流转至审核人；审核通过后流转至批准人；批准通过后样品状态进入留样。</p>
              <p>编制人、审核人不通过时退回修改；批准人可选择批准留样或不同意转销样。当前登录用户由登录页账号决定，不允许在审批页面手动切换角色。</p>
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
                <label>总代表数量/t<input v-model="samplingForm.representativeQuantity" /></label>
                <label>性质<input v-model="samplingForm.nature" /></label>
                <label>产地<input v-model="samplingForm.origin" /></label>
                <label>生产/进口年限<input v-model="samplingForm.productionYear" /></label>
                <label v-if="samplingSource === 'internal' && samplingReason !== '入库前检验'">入仓/罐时间<input v-model="samplingForm.storageDate" /></label>
                <label v-if="samplingSource === 'internal' && samplingReason === '出库检验'">包装/散装<select v-model="samplingForm.packageType"><option>散装</option><option>包装</option></select></label>
                <label>样品数量<input v-model="samplingForm.sampleCount" /></label>
                <label>样品单位<select><option>kg</option><option>L</option></select></label>
                <label>{{ samplingSource === 'external' ? '每仓留样份数' : '留样份数' }}<input v-model="samplingForm.retainCount" /></label>
                <label>{{ samplingSource === 'external' ? '每仓检验样份数' : '检验样份数' }}<input v-model="samplingForm.inspectionCount" /></label>
                <label>{{ samplingSource === 'external' ? '每仓样品份数' : '样品份数' }}<input v-model="samplingForm.totalCopies" /></label>
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
                  <tr><th>样品数量</th><td>{{ previewSamplingRecord.sampleCount }}</td><th>代表数量/t</th><td>{{ previewSamplingRecord.representativeQuantity ?? '100.000' }}</td><th>性质</th><td>{{ previewSamplingRecord.nature ?? '中央储备粮' }}</td></tr>
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
                <span>样品状态</span>
                <label v-for="status in activeLabelStatuses" :key="status">
                  <input v-model="selectedLabelStatus" type="radio" :value="status" :disabled="!editableLabelStatuses.includes(status)" @change="syncLabelStatus" />
                  {{ status }}
                </label>
                <button type="button" @click="printPage"><Printer :size="14" /> 打印</button>
              </div>
              <div class="print-sample-label">
                <h2>样品标签</h2>
                <div class="label-row two">
                  <div><span>名称：</span><b>{{ selectedLabelRecord.sampleName }}</b></div>
                  <div><span>编号：</span><b>{{ selectedLabelRecord.sampleNo }}</b></div>
                </div>
                <div class="label-row two">
                  <div><span>来样日期：</span><b>{{ selectedLabelRecord.receiveTime }}</b></div>
                  <div><span>规格：</span><b>{{ selectedLabelRecord.sampleCount }}</b></div>
                </div>
                <div class="label-checks">
                  <label v-for="status in activeLabelStatuses" :key="status">
                    <span :class="{ checked: selectedLabelStatus === status }"></span>
                    {{ status }}
                  </label>
                </div>
                <p>中央储备粮镇江直属库有限公司（承储）</p>
              </div>
            </div>
          </section>
        </div>

        <div v-if="showReportForm" class="process-dialog-mask" @click.self="showReportForm = false">
          <section class="report-form-dialog">
            <div class="process-dialog-head">
              <div><h2>录入检验结果</h2><span>{{ selectedReportRecord.sampleNo }}</span></div>
              <button type="button" @click="showReportForm = false">关闭</button>
            </div>
            <div class="report-form-body">
              <section class="report-section">
                <h3>基础信息</h3>
                <div class="report-base-grid">
                  <label>样品编号<input :value="selectedReportRecord.sampleNo" readonly /></label>
                  <label>品种<input :value="selectedReportRecord.grainType" readonly /></label>
                  <label>送样单位<input :value="selectedReportRecord.company" readonly /></label>
                  <label>储备性质<input value="中央储备粮" readonly /></label>
                  <label>送样日期<input :value="selectedReportRecord.sendTime ?? selectedReportRecord.receiveTime" readonly /></label>
                  <label>送样人<input :value="selectedReportRecord.sender ?? '—'" readonly /></label>
                  <label>样品规格<input :value="selectedReportRecord.sampleCount" readonly /></label>
                  <label>样品份数<input value="4" readonly /></label>
                  <label>仓（罐）号<input :value="selectedReportRecord.warehouseNo" readonly /></label>
                  <label>代表数量(吨)<input value="100.000" readonly /></label>
                  <label>产地<input value="江苏" readonly /></label>
                  <label>质检报告编号<input :value="selectedReportRecord.reportNo ?? '系统提交后生成'" readonly /></label>
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
                  <thead><tr><th>检验项目</th><th>检验依据</th><th>合同要求/检验标准值</th><th>检验结果</th><th>单项判定</th><th>检验人</th></tr></thead>
                  <tbody>
                    <tr v-for="item in qualityLevelItems" :key="item">
                      <td>{{ item }}</td>
                      <td><select><option>请选择</option><option>GB/T 5494-2019</option><option>GB 5009.3-2016</option></select></td>
                      <td><input placeholder="请输入" /></td>
                      <td><input :value="reportResultValue(item, 'detectValue')" placeholder="请输入" @input="updateReportResult(item, 'detectValue', ($event.target as HTMLInputElement).value)" /></td>
                      <td><select :value="reportResultValue(item, 'judgement') || '合格'" @change="updateReportResult(item, 'judgement', ($event.target as HTMLSelectElement).value)"><option>合格</option><option>不合格</option></select></td>
                      <td><select :value="reportResultValue(item, 'inspector') || '质检员A'" @change="updateReportResult(item, 'inspector', ($event.target as HTMLSelectElement).value)"><option>质检员A</option><option>张三</option><option>李四</option></select></td>
                    </tr>
                  </tbody>
                </table>
              </section>

              <section class="report-section">
                <h3>卫生指标</h3>
                <table class="inspection-result-table compact">
                  <thead><tr><th>检验项目</th><th>检验依据</th><th>合同要求/检验标准值</th><th>检验结果</th><th>单项判定</th><th>检验人</th></tr></thead>
                  <tbody>
                    <tr v-for="item in healthIndicatorItems" :key="item">
                      <td>{{ item }}</td><td><select><option>GB 2761-2017</option></select></td><td><input placeholder="请输入" /></td><td><input :value="reportResultValue(item, 'detectValue')" placeholder="请输入" @input="updateReportResult(item, 'detectValue', ($event.target as HTMLInputElement).value)" /></td><td><select :value="reportResultValue(item, 'judgement') || '合格'" @change="updateReportResult(item, 'judgement', ($event.target as HTMLSelectElement).value)"><option>合格</option><option>不合格</option></select></td><td><select :value="reportResultValue(item, 'inspector') || '质检员A'" @change="updateReportResult(item, 'inspector', ($event.target as HTMLSelectElement).value)"><option>质检员A</option><option>张三</option><option>李四</option></select></td>
                    </tr>
                  </tbody>
                </table>
                <div class="health-extra-grid">
                  <label>等级<select><option>请选择</option><option>一等</option><option>二等</option></select></label>
                  <label>是否达标<select><option>请选择</option><option>是</option><option>否</option></select></label>
                  <label class="full">检测结论<textarea placeholder="请输入检测结论"></textarea></label>
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

        <div v-if="showReportView" class="process-dialog-mask" @click.self="showReportView = false">
          <section class="report-form-dialog readonly-report-dialog">
            <div class="process-dialog-head"><div><h2>质检报告详情</h2><span>{{ selectedReportRecord.reportNo ?? '未生成编号' }}</span></div><button type="button" @click="showReportView = false">关闭</button></div>
            <div class="report-form-body readonly-report-body">
              <section class="report-section">
                <h3>基础信息</h3>
                <div class="report-base-grid">
                  <label>样品编号<input :value="selectedReportRecord.sampleNo" readonly /></label>
                  <label>品种<input :value="selectedReportRecord.grainType" readonly /></label>
                  <label>送样单位<input :value="selectedReportRecord.company" readonly /></label>
                  <label>储备性质<input value="中央储备粮" readonly /></label>
                  <label>送样日期<input :value="selectedReportRecord.sendTime ?? selectedReportRecord.receiveTime" readonly /></label>
                  <label>送样人<input :value="selectedReportRecord.sender ?? '—'" readonly /></label>
                  <label>样品规格<input :value="selectedReportRecord.sampleCount" readonly /></label>
                  <label>样品份数<input value="4" readonly /></label>
                  <label>仓（罐）号<input :value="selectedReportRecord.warehouseNo" readonly /></label>
                  <label>代表数量(吨)<input value="100.000" readonly /></label>
                  <label>产地<input value="江苏" readonly /></label>
                  <label>质检报告编号<input :value="selectedReportRecord.reportNo ?? '系统提交后生成'" readonly /></label>
                  <label>检验事由<input :value="selectedReportRecord.reason" readonly /></label>
                  <label>记录人<input value="系统记录员" readonly /></label>
                  <label>检测类别<input :value="selectedReportRecord.reportMeta?.category ?? '监督检验'" readonly /></label>
                  <label>编制人<input :value="selectedReportRecord.reportMeta?.compiler ?? '王帅'" readonly /></label>
                  <label>审核人<input :value="selectedReportRecord.reportMeta?.reviewer ?? '李审核'" readonly /></label>
                  <label>批准人<input :value="selectedReportRecord.reportMeta?.approver ?? '赵批准'" readonly /></label>
                  <label class="full">备注<textarea :value="selectedReportRecord.reportMeta?.remark || '质检报告详情与录入内容一致，仅供查看，不能编辑。'" readonly></textarea></label>
                </div>
              </section>

              <section class="report-section">
                <h3>质量等级</h3>
                <table class="inspection-result-table">
                  <thead><tr><th>检验项目</th><th>检验依据</th><th>合同要求/检验标准值</th><th>检验结果</th><th>单项判定</th><th>检验人</th></tr></thead>
                  <tbody>
                    <tr v-for="item in qualityLevelItems" :key="item" :class="{ 'unqualified-row': qualityResultValue(selectedReportRecord, item) === '不合格' }">
                      <td>{{ item }}</td><td>GB/T 5494-2019</td><td>符合中央储备粮质量标准</td><td>{{ qualityDetectValue(selectedReportRecord, item) }}</td><td>{{ qualityResultValue(selectedReportRecord, item) }}</td><td>{{ selectedReportRecord.reportResults?.[item]?.inspector ?? '质检员A' }}</td>
                    </tr>
                  </tbody>
                </table>
              </section>

              <section class="report-section">
                <h3>卫生指标</h3>
                <table class="inspection-result-table compact">
                  <thead><tr><th>检验项目</th><th>检验依据</th><th>合同要求/检验标准值</th><th>检验结果</th><th>单项判定</th><th>检验人</th></tr></thead>
                  <tbody>
                    <tr v-for="item in healthIndicatorItems" :key="item" :class="{ 'unqualified-row': healthResultValue(selectedReportRecord, item) === '不合格' }">
                      <td>{{ item }}</td><td>GB 2761-2017</td><td>符合限量要求</td><td>{{ healthDetectValue(selectedReportRecord, item) }}</td><td>{{ healthResultValue(selectedReportRecord, item) }}</td><td>{{ selectedReportRecord.reportResults?.[item]?.inspector ?? '质检员A' }}</td>
                    </tr>
                  </tbody>
                </table>
                <div class="health-extra-grid">
                  <label>等级<input :value="reportHasUnqualified(selectedReportRecord) ? '不合格' : '一等'" :class="{ 'unqualified-field': reportHasUnqualified(selectedReportRecord) }" readonly /></label>
                  <label>是否达标<input :value="reportHasUnqualified(selectedReportRecord) ? '否' : '是'" :class="{ 'unqualified-field': reportHasUnqualified(selectedReportRecord) }" readonly /></label>
                  <label class="full">检测结论<textarea :value="reportHasUnqualified(selectedReportRecord) ? '该批样品存在不合格指标，不符合质量要求，建议进入销样处置流程。' : '该批样品各项指标符合质量要求。'" :class="{ 'unqualified-field': reportHasUnqualified(selectedReportRecord) }" readonly></textarea></label>
                </div>
              </section>
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
                <div class="history-mode"><span>历史审批意见</span><label><input type="radio" checked /> 正序</label><label><input type="radio" /> 倒序</label></div>
                <div class="approval-timeline">
                  <div v-for="item in [...(selectedReportRecord.approvalHistory ?? [])].reverse()" :key="`${item.title}-${item.time}`" class="approval-timeline-item">
                    <i></i><b>{{ item.title }}</b><span>{{ item.user }} {{ item.time }}</span>
                  </div>
                </div>
                <div v-if="canApproveReport(selectedReportRecord)" class="approval-actions inline">
                  <div>当前节点：{{ selectedReportRecord.approvalStep }}，当前用户：{{ currentLoginUser.name }}（{{ currentLoginUser.role }}）</div>
                  <button type="button" @click="approveReport(selectedReportRecord.sampleNo, 'pass')"><CheckCircle2 :size="13" /> 审批通过/同意</button>
                  <button v-if="currentLoginUser.role !== '编制人'" type="button" @click="approveReport(selectedReportRecord.sampleNo, 'return')"><CornerDownLeft :size="13" /> 不同意退回修改</button>
                  <button v-if="currentLoginUser.role === '批准人'" type="button" @click="approveReport(selectedReportRecord.sampleNo, 'destroy')"><Trash2 :size="13" /> 不同意转销样</button>
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
              </div>
              <div class="unqualified-detail-wrap">
                <table class="unqualified-detail-table">
                  <colgroup><col class="unqualified-index-col" /><col /><col /><col /><col /></colgroup>
                  <thead><tr><th>序号</th><th>不合格项目</th><th>检测结果</th><th>判定</th><th>处置建议</th></tr></thead>
                  <tbody>
                    <tr v-for="(item, index) in reportUnqualifiedItems(selectedUnqualifiedRecord)" :key="item">
                      <td>{{ index + 1 }}</td><td class="unqualified-text">{{ item }}</td><td>{{ qualityLevelItems.includes(item) ? qualityDetectValue(selectedUnqualifiedRecord, item) : healthDetectValue(selectedUnqualifiedRecord, item) }}</td><td><span class="warning-level-badge alarm">不合格</span></td><td>{{ reportUnqualifiedCount(selectedUnqualifiedRecord) >= 3 ? '进入质量报警处置，建议销样或复检' : '进入质量预警，建议复核检测' }}</td>
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
              <label>送样人名称<input v-model="sendSampleForm.sender" /></label>
              <label>送样时间<input v-model="sendSampleForm.sendTime" type="datetime-local" /></label>
              <button type="button" @click="confirmSendSample">确认送样</button>
            </div>
          </section>
        </div>

        <div v-if="showReceiveSample" class="process-dialog-mask" @click.self="showReceiveSample = false">
          <section class="send-sample-dialog">
            <div class="process-dialog-head"><div><h2>收样登记</h2><span>{{ selectedSamplingRecord.orderNo }}</span></div><button type="button" @click="showReceiveSample = false">关闭</button></div>
            <div class="send-form">
              <label>收样人名称<input v-model="receiveSampleForm.receiver" /></label>
              <label>收样时间<input v-model="receiveSampleForm.receiveTime" type="datetime-local" /></label>
              <button type="button" @click="confirmReceiveSample">确认收样</button>
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
              <label>审批角色<select v-model="userForm.role"><option>编制人</option><option>审核人</option><option>批准人</option><option>质检员</option></select></label>
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
